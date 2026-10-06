# Batteries Cost-Sync Adapter Interface

**Date:** 6 October 2026\
**Status:** Working draft for ISS-01 and ISS-02; source-verified, no runtime proof\
**Bead:** `netspe-scr.29`

## Purpose and evidence limits

This is the interface the enterprise gateway mocks for copying Clearinghouse
Batteries usage into the engine store. It certifies the pull path assumed by
[ISS-01](https://github.com/Cloud-SPE/Network-Engineering-SPE/issues/7) and
[ISS-02](https://github.com/Cloud-SPE/Network-Engineering-SPE/issues/8). The
adapter syncs rows. It does not query or select them. Per-job cost, the
status-to-cost mapping, and selection stay in the engine's Postgres store
adapter.

Reviewed against source, not a running signer, Kafka, and Batteries deployment:

| Component | Revision | Where it stands |
| --- | --- | --- |
| [`livepeer/clearinghouse-batteries`](https://github.com/livepeer/clearinghouse-batteries) | `501c1ed` on `main` | Upstream. Usage attribution and the cursor are [#19](https://github.com/livepeer/clearinghouse-batteries/pull/19) `430b68a` and [#20](https://github.com/livepeer/clearinghouse-batteries/pull/20) `e8842e2` |
| go-livepeer | `773734d` (v0.9.3) | Upstream. `server/remote_signer.go` puts `manifest_id` on the `create_signed_ticket` event |
| Python gateway SDK | [`5868d81`](https://github.com/livepeer/livepeer-python-gateway/commit/5868d81d5bfcad305af96c57c679acee79e07edb) on `feat/call-runner-manifest-id` | Draft [livepeer-python-gateway#71](https://github.com/livepeer/livepeer-python-gateway/pull/71) against upstream `main`. Not merged. A streamed or failed paid call reports the manifest id it was charged under. Errors and runner rejections carry `payment_sent`, and a rejection is classified `capacity`, `unreachable`, or `other` |

`go test ./...` passed at `501c1ed` on 6 October 2026. That is source evidence.
It is not an end-to-end proof that a ticket becomes a usage row.

The [builder-layer proposal](2026-10-01-Builder-Layer-Abstraction-and-Two-Application-Review.md)
assumed a Batteries cost read of `GET /v1/cost/events` and
`GET /v1/cost/manifests/{manifest_id}`. Those routes are not on Batteries
`main`. `GET /v1/usage` is the pull path this interface uses. The imported
proposal text is unchanged.

## Wire shape

`GET /v1/usage` on the management listener. The caller sends
`Livepeer-Clearinghouse-Token` for a `management` credential with `usage.read`.
The route allows only `grant_id`, `allocation_id`, `manifest_id`, `limit`, and
`cursor`. Unknown parameters, repeated parameters, and empty values return `400`.
`limit` defaults to 100 and must be an integer from 1 through 1,000.

The sync uses no id filters. Filtering by manifest or allocation is the store
adapter's job after the rows are copied. A filtered tail would also drop usage
that has no payment session: Batteries includes those rows only in an unfiltered
list.

```json
{
  "items": [
    {
      "id": "01J0USAGE",
      "event_id": "evt-1",
      "topic": "create_signed_ticket",
      "partition": 0,
      "offset": 42,
      "status": "applied",
      "error": "",
      "computed_fee_eth": "0.00000000000000001",
      "computed_fee_usd": "0.25",
      "created_at_ms": 1759766400000,
      "payment_session_id": "01J0SESSION",
      "request_id": "req-1",
      "pipeline": "fixed",
      "manifest_id": "manifest-1",
      "allocation_id": "01J0ALLOC",
      "currency": "usd"
    }
  ],
  "next_cursor": ""
}
```

Amounts are decimal strings. `created_at_ms` is Unix milliseconds. A row with no
payment session has null `payment_session_id`, `allocation_id`, `currency`, and
often null `manifest_id`. The stored column is `computed_fee_wei`; the
management API renames every `_wei` amount to `_eth` before it writes the JSON.
`computed_fee_usd` keeps its name. `topic`, `partition`, `offset`, and `error`
are on the wire and are not part of the contract below.

`next_cursor` is an opaque string bound to this route and to the filters that
produced it. Sending it with a different filter set returns `400` with
`cursor does not match resource or filters`. An empty `next_cursor` means this
page is the last one. Batteries does not return a cursor that resumes after the
last row.

## Contract

Frozen pydantic v2 models, the same rule as the
[builder-layer proposal](2026-10-01-Builder-Layer-Abstraction-and-Two-Application-Review.md):
the in-process class is the JSON schema. `Decimal` serializes as a string.
Times are UTC.

```python
class UsageRow(Contract):
    id: str                          # usage_events.id; the dedupe key
    event_id: str | None
    status: Literal["applied", "quarantined", "ignored", "duplicate"]
    manifest_id: str | None
    allocation_id: str | None
    payment_session_id: str | None
    request_id: str | None
    pipeline: str | None
    computed_fee_eth: Decimal | None # JSON name of the computed_fee_wei column
    computed_fee_usd: Decimal | None
    currency: str | None             # "usd" or "eth" when a session matched
    created_at: datetime             # from created_at_ms

class UsagePage(Contract):
    items: tuple[UsageRow, ...]
    next_cursor: str                 # "" when this page is the last

class SyncCheckpoint(Contract):
    resume_cursor: str | None        # cursor that fetched the last page; None starts at the beginning
    filters: Mapping[str, str]       # empty for the unfiltered tail; a cursor is bound to these

class SyncReport(Contract):
    pages: int
    rows_seen: int
    rows_new: int

class UsageSource(Protocol):         # BatteriesUsageSource implements this
    async def page(self, cursor: str | None, limit: int = 1000) -> UsagePage: ...

class CostSyncStore(Protocol):       # the Postgres store adapter implements this
    async def checkpoint(self) -> SyncCheckpoint | None: ...
    async def apply(self, rows: Sequence[UsageRow], checkpoint: SyncCheckpoint) -> int: ...

class CostSyncWorker:
    async def sync_once(self) -> SyncReport: ...
```

`CostSyncStore.apply` upserts by `UsageRow.id` and writes the checkpoint in the
same transaction. It returns how many of those ids were new. `CostSyncWorker`
is constructed with one `UsageSource`, one `CostSyncStore`, and the filter set
that source was built with. The mocked enterprise gateway uses an empty filter
set.

## Behavior

1. Read the checkpoint. If it is missing, or its `filters` differ from the
   worker's filter set, start with `resume_cursor = None`.
2. Fetch a page with that cursor. If Batteries returns `400` because the cursor
   does not match, clear the checkpoint and fetch once more from the start.
   Any other error leaves the checkpoint unchanged.
3. Choose the checkpoint to store with the page. If `next_cursor` is nonempty,
   store that value: the next poll continues forward. If `next_cursor` is
   empty, store the cursor that fetched this page, so the next poll re-reads
   the last page. A first page that is also the last page stores `None`.
4. Upsert the rows and that checkpoint in one transaction. A crash after the
   fetch and before the commit leaves the previous checkpoint, and the next
   `sync_once` fetches the same page again.
5. Stop when `next_cursor` is empty. Otherwise repeat from step 2 with
   `next_cursor`.

The store keeps every status. Only `applied` rows are observed cost. A
`quarantined` row with no session is still stored; it must not become a zero
cost. Rows are never updated in place upstream, so a later change for the same
manifest arrives as a new `id`. Dedupe is on `id`, not on `manifest_id`.
`manifest_id` is not unique. The store joins cost on `(allocation_id, manifest_id)`.

Re-reading the last page is what makes an empty `next_cursor` usable. New usage
gets a higher insertion sequence, so it appears after the cursor that fetched
the previous last page. Rows already stored are dropped by the upsert.

## Test fakes

`ScriptedUsageSource` returns a fixed list of `UsagePage` values.
`MemoryCostSyncStore` keeps rows in a dict keyed by `id` and keeps one
checkpoint. The suite covers:

- A last page with `next_cursor == ""`, a second `sync_once`, and no duplicate
  rows counted as new.
- A new row arriving after that second poll, picked up by re-reading the last
  page.
- A `quarantined` row with null `allocation_id` and `manifest_id`, stored and
  excluded from observed cost.
- A `400` cursor mismatch, which clears the checkpoint and syncs from the start.
- A crash after a page is fetched and before `apply` commits: the next
  `sync_once` fetches that page again and `rows_new` is zero for the rows
  already stored.

## Decisions

- Sync the unfiltered usage list. Selection by allocation, manifest, and status
  happens in Postgres after the copy. A filtered Batteries query cannot see
  rows that have no payment session, and its cursor cannot be reused if the
  filter changes.
- Treat an empty `next_cursor` as "re-read the last page", not as "forget the
  cursor". Restarting from the beginning is only acceptable for a trivial history.
- Keep non-`applied` rows. Dropping them in the adapter would make a missing
  fee look like a zero fee.
- Key observed cost by `(allocation_id, manifest_id)`. One manifest can appear
  on more than one allocation, and one job can have more than one applied row.
- Leave signer delivery as a gap. A durable copy of `/v1/usage` cannot restore
  an event go-livepeer dropped before Kafka.

## Not in this interface

Job cost queries, `JobCost` assembly, and which attempt a manifest belongs to.
Reads scoped to one credential (hosted-operator mode). The signer's
best-effort Kafka publish. Engine job idempotency (`netspe-scr.8`), which is
a different key from Batteries' `Idempotency-Key`.

## Implementation

- `BatteriesUsageSource` and `CostSyncWorker` implement this contract over
  `GET /v1/usage`. The Postgres `CostSyncStore` owns querying and selection.
- `netspe-cz5.16` is the `BatteriesProvider` client. This sync is the cost half
  of that client.
- `netspe-scr.14` records the maintainer disposition for the usage route.
  `netspe-scr.8` stays open.
