# Clearinghouse Configuration

**Status:** Draft for review
**Updated:** 25 September 2026
**Context:** [Usage event export](usage-event-export.md), [provisioning modes](payment-provisioning-modes.md)

Clearinghouse Batteries configures payment operation. It does not configure enterprise login. The flags below are the `serve` surface in the current Batteries tree. The export topic is the only addition this set of drafts asks for, and it needs maintainer agreement before it is implemented.

No Cloud SPE decision record accepts a change to these flags yet.

## Current serve flags

| Flag | Role |
| --- | --- |
| `--db-path` | SQLite ledger on a local filesystem |
| `--enable-auth-webhook` | Signer authorization HTTP server. `:port` binds to loopback |
| `--unsafe-http-bind` | Required for a non-loopback webhook bind |
| `--webhook-token-file` | `Livepeer-Clearinghouse-Token` shared with the signer. Direct secret flags are rejected |
| `--enable-kafka` | Embedded broker. Incompatible with `--kafka-brokers` |
| `--kafka-brokers` | External plaintext brokers for accounting without the embedded broker |
| `--kafka-bind` | Embedded broker address. Loopback or private IP |
| `--kafka-topic` | Signer signing topic. Default `livepeer-signing` |
| `--enable-accounting` | Ledger consumer. One accounting process per database |
| `--enable-onchain-listener` | Settlement and escrow listener. One listener per database |
| `--rpc-url-file`, `--chain-id`, `--ticket-broker`, `--signer-addresses`, `--start-block`, `--confirmations`, `--poll-interval`, `--block-batch-size`, `--reorg-lookback` | On-chain listener settings |

Configuration precedence is flags, then environment, then the configuration file, then defaults. Webhook and RPC secrets come from the environment or a mounted file.

The webhook stays on loopback unless a TLS proxy terminates in front of it, matching Batteries `SECURITY.md`. Kafka stays on a trusted network. Plaintext broker connections are an operator concern, not an enterprise API.

Management of grants, allocations, keys, sessions, usage, ledger, settlement, and escrow remains the CLI. The maintainer's admin HTTP server is not a `serve` flag and is not part of the export-topic ask. `usage list` returns id, event id, topic, offset, status, error, fee, and created time. Pipeline, request id, session, billable seconds, and pixels stay in the table and are omitted from that CLI view. Enterprises use the export topic in the [usage draft](usage-event-export.md) rather than polling this database or this CLI.

## Export topic

When the maintainer accepts the export, `serve` gains one optional flag:

```text
--usage-export-topic
```

Unset, Batteries behaves as it does today: ingest, ledger, and no outbound usage publish. Set, each applied `create_signed_ticket` is also published to that topic as a CloudEvent. The flag uses the same broker as accounting (`--enable-kafka` or `--kafka-brokers`). It does not open a second cluster and it does not add an enterprise identity provider, a retail price, or a billing-vendor URL.

Funding idempotency is a separate maintainer ask. Today each fund operation mints a new ledger idempotency key (`fund:` plus a fresh id), so a retried fund posts twice. Caller-supplied funding idempotency is part of the [open provisioning gap](payment-provisioning-modes.md#open-gap-allocation-provisioning-and-funding) and is left to that later design. This draft does not add a fund-idempotency flag.

## Modes and flags

Standalone, self-operated enterprise, and hosted payment deployments use this same flag set. The gateway's access-adapter mode, signer URL, and discovery URL are gateway configuration. The allocation key is a vault secret on the enterprise app's authentication server in enterprise mode, or gateway configuration in standalone mode. They are not Batteries flags.

Hard spending reservation, a public management HTTP API, and an outbox table are out of this flag change. Programmatic grant and allocation management waits on the maintainer's admin HTTP server and is not requested here. The export topic is the outbound path. Authorization continues to check a positive available balance and does not reserve the next ticket.

## Decisions

- Existing serve flags stay the operational surface.
- The only proposed flag is `--usage-export-topic`, default off.
- Enterprise authentication is configured on the gateway, not on Batteries.
- SQLite is not given a new remote read API for billing.

## Work this design implies

`netspe-cz5.11` is the upstream `--usage-export-topic` patch. It waits on maintainer agreement. It does not wait on the admin HTTP server. Every other `serve` flag stays unchanged.
