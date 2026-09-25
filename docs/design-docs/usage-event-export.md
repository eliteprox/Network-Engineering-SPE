# Usage Event Export

**Status:** Draft for review
**Updated:** 25 September 2026
**Context:** [Clearinghouse configuration](clearinghouse-configuration.md), [provisioning modes](payment-provisioning-modes.md), [architecture companion](open-builder-architecture-and-sequences.md#persistence-and-accounting-authority)

Enterprises receive wholesale network usage from a Kafka topic. They do not read `clearinghouse.db`. Each event is charged to the enterprise's single wholesale allocation, as described in the [provisioning draft](payment-provisioning-modes.md#wholesale-accounting-model). Batteries does not know end users. Retail rating, subscriptions, invoices, and per-user attribution stay in the enterprise application and billing system.

No Cloud SPE decision record accepts this contract yet. Publishing an export topic is a Clearinghouse change and needs maintainer agreement.

## Source event

The remote signer publishes `create_signed_ticket` envelopes to the signing topic. Clearinghouse Batteries consumes that topic in its own consumer group and posts the ledger in the same transaction as the checkpoint. The current `SignedTicketEvent` stores session, auth id, app, pipeline, request id, orchestrator, PM session, computed fee in wei, sequence, ticket count, timestamps, billable seconds, and pixels.

[go-livepeer#4095](https://github.com/livepeer/go-livepeer/pull/4095) is open. It adds `price_usd` on remote discovery and `computed_fee_usd` plus `payer_address` on the Kafka event. Encoding/json ignores unknown fields, so a merge keeps current ingest working. Those fields are stored on the export event when present. They appear in SQLite only if the ingest struct is extended later. The export copies them from the raw signer payload at apply time.

## Export point

When Batteries marks an event `applied`, it publishes one [CloudEvents 1.0](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/spec.md) record to an export topic. The allocation lookup already performed during ingest supplies the routing key. Quarantined, duplicate, and ignored events remain in SQLite and are omitted from the export.

The export record uses the signer event id as CloudEvents `id`, so consumers can deduplicate. Suggested attributes:

| Attribute | Value |
| --- | --- |
| `specversion` | `1.0` |
| `type` | `livepeer.clearinghouse.signed_usage` |
| `source` | Stable URI of this Clearinghouse deployment |
| `id` | Signer envelope id |
| `time` | Event end timestamp |
| `subject` | Opaque `enterprise_id` from allocation metadata |
| `data` | Fee in wei, optional `computed_fee_usd` and `payer_address`, pipeline, `manifest_id`, request id, auth id, allocation id, quantity fields present on the signer event |

`enterprise_id` is an opaque string written into allocation metadata at provisioning. It selects which enterprise receives the event. It is not a customer id, a subscription id, or a retail price. Several enterprises can share one signer and one Batteries database. Each allocation carries its own `enterprise_id`.

The signer's `request_id` is a fresh random id for each payment call, so it cannot identify a job. The signer event also carries `manifest_id`, which equals the `session_id` the Python SDK returns for a Live Runner call. Batteries does not parse `manifest_id` today; the export copies it from the raw signer payload. The engine records `manifest_id` on each attempt, and the enterprise joins its own job and user records to export events on that key. That join is how an enterprise attributes wholesale cost to its end users.

The engine may also keep a job-correlated projection for `GET /v1/usage`. That projection joins export or ledger evidence to job and attempt ids through `manifest_id`. It is not a second network ledger. Missing or late fees stay pending. Replaying the projection cannot recreate a signer event that was never published.

## Billing connectors

A connector outside Batteries translates the CloudEvent into the billing product the enterprise runs. Batteries does not call these APIs.

| Product | Ingest the product accepts | Connector responsibility |
| --- | --- | --- |
| OpenMeter and Kong Metering & Billing | `POST /api/v1/events`, or Konnect `POST /v3/openmeter/events`, as `application/cloudevents+json`. Batches use `application/cloudevents-batch+json`. Required fields include `id`, `source`, `type`, and `subject`. Documented cloud ingest publishes into that product's own Kafka. It does not subscribe to an operator-chosen topic. | Set `subject` to the enterprise customer, resolved from the enterprise's job records by `manifest_id`. Keep `id` equal to the signer event id. |
| Lago | `POST /api/v1/events`, or Lago's own Kafka topic when the ClickHouse event store is enabled. The body is `transaction_id`, `external_subscription_id`, `code`, an explicit `timestamp`, and `properties`. CloudEvents ingestion was declined in [getlago/lago-api#1986](https://github.com/getlago/lago-api/issues/1986). On the ClickHouse store, deduplication uses `transaction_id` and `timestamp` together, so a retry resends the original payload. | Map `transaction_id` from the signer event id. Fill `external_subscription_id` and billable-metric `code` from the enterprise catalog. |
| Kill Bill | Aviate raw events use `POST /plugins/aviate-plugin/v1/metering/billing/{accountId}` with `billingMeterCode`, `subscriptionId`, `timestamp` at second precision, and `value`. The core usage API expects pre-aggregated rolls. There is no CloudEvents ingest. A community Kafka plugin expects Kill Bill's own usage JSON. | Fill account, subscription, and meter code. Aggregate before calling the core usage API when Aviate is not the sink. |

OpenMeter can accept the canonical CloudEvent with a subject mapping. Lago and Kill Bill each need a schema adapter. The retail identifiers those products require are added in the connector.

## Decisions

- The read path for enterprises is the export topic, written on apply.
- SQLite remains the ledger and the audit of quarantined events. It is not the enterprise query API.
- `enterprise_id` on the allocation is the multi-tenant routing key. End-user attribution is not in the export.
- `manifest_id` is the job correlation key. The signer's `request_id` is not.
- `computed_fee_usd` and `payer_address` ride the export when #4095 has merged, and are optional until then.
- One CloudEvent schema feeds every connector. Product-specific retail fields are added outside Batteries.

## Work this design implies

`netspe-cz5.11` is the export-topic publish and waits on maintainer agreement, not on the admin HTTP server. `netspe-cz5.12` is the CloudEvent schema fixture. `netspe-cz5.13` is the OpenMeter HTTP ingest connector. `netspe-cz5.14` and `netspe-cz5.15` are the Lago and Kill Bill adapters.
