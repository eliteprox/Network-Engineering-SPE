# Payment Provisioning Modes

**Status:** Draft for review
**Updated:** 6 October 2026
**Context:** [Enterprise authorization server](enterprise-authorization-server.md), [Batteries management integration](batteries-management-integration.md), [usage event export](usage-event-export.md), [architecture companion](open-builder-architecture-and-sequences.md#payment-operation-choices)

This draft maps how much of the Clearinghouse grant model a deployment exposes. It uses the payment modes from the architecture companion and the access modes from the [authorization draft](enterprise-authorization-server.md). Those choices are independent: the enterprise app's authentication server can sit on a self-operated ledger or on a hosted payment operator.

No Cloud SPE decision record accepts this contract yet.

## Grant, allocation, and key

A grant is the customer budget. Batteries was originally designed for community grants, so one grant is one customer. An allocation is a partition of that grant, with its own balance, status, and optional time window. An operator can keep CI on one slice and production on another. Several API keys can hang off one allocation so a key can be rotated and revoked without a new budget.

`POST /v1/signer/authorize` is a signer-to-Clearinghouse call. The signer sends `Livepeer-Clearinghouse-Token`. The body carries the gateway's allocation key as `Authorization: Bearer`. Clearinghouse hashes that key and loads `api_keys.allocation_id`. The request does not carry an `allocation_id`. A matching active key with an active grant, an allocation in `active` status, a positive available balance, and a coherent session binding returns HTTP 200 and JSON `status` 200 plus `auth_id`. The session binds state id, app, payment type, orchestrator, and key.

Public job routes do not create keys. Revocation stops later authorizations. Rotating or revoking the key ends payment sessions bound to it; the gateway must load the replacement key. Tickets already signed can still arrive and post. Creating or funding an allocation does not fund signer escrow. Escrow funding stays an operator action on the signer wallet.

Grant, allocation, and key operations are CLI commands and, since Batteries `a2ed175`, management HTTP routes under a service credential. The CLI is not caller-idempotent: each fund inserts a new `fund:` ledger key, and each create mints a new id. On the management API, allocation creation, API-key creation, and grant and allocation funding accept `Idempotency-Key` (Batteries [#9](https://github.com/livepeer/clearinghouse-batteries/pull/9)). Grant creation does not. A matching retry replays the original success, including the API-key secret. A reused key with a different body returns `409`.

The enterprise app's authentication server stores the `lpg_` secret as a vault entry scoped to `sub`. The enterprise user row may store `allocation_id` so the operator can fund the same slice later. The client never receives the key. See the [authorization draft](enterprise-authorization-server.md#user-scoped-vault).

## Wholesale accounting model

Batteries has no user, customer, or account record. Its records are grants and allocations (budgets), API keys (gateway credentials), payment sessions (one per signer payment state), and signed-ticket usage events. It keeps an event-level audit trail, but that trail rolls up to the allocation balance. Record volume grows with traffic, not with the number of end users.

Enterprise deployments therefore start with one wholesale network-credit grant per enterprise, with engine-managed allocations under it ([Phase A](batteries-management-integration.md#phase-a-single-tenant-reseller)). Batteries authorizes signing while that allocation has a positive available balance, debits each signed ticket, and reports network cost as expected value in wei. It does not know which end user caused a cost. Every `sub` may resolve to that one vault key.

The enterprise application owns everything below that allocation:

- end-user identities, allowances, subscriptions, and per-user spend limits
- retail rating, pricing, markup, invoices, and payment collection
- per-user or per-job attribution, joined from its own job records to the [usage export](usage-event-export.md)

Because every end user shares the allocation, Clearinghouse cannot stop one user from exhausting the enterprise's budget. It returns decision status 402 only when the whole allocation is exhausted. The gateway or enterprise application enforces per-user limits, including during metered sessions and not only before invocation. A separate allocation per end customer or device is possible through the management API and belongs to the [multi-tenant phase](batteries-management-integration.md#phase-b-multi-tenant). It is still a budget label rather than a user record.

## Standalone

The operator uses the CLI: one grant, one allocation, one API key, installed in gateway configuration. Callers authenticate to the gateway with an operator-issued gateway API key, or anonymously only on a local test profile that is not a public deployment. Callers never see grant ids. The engine reports network cost for that single allocation. `enterprise_id` metadata may be omitted when there is one tenant.

## Self-operated enterprise

The enterprise operates its own Batteries and signer. One wholesale allocation per enterprise is provisioned through the management API, or with the CLI on the Batteries host, with `enterprise_id` set in allocation metadata at creation for the [usage export](usage-event-export.md). The operator installs the one-time allocation key in the vault on the enterprise app's authentication server. End-user routes and MCP tools do not create, fund, or read allocations.

## Hosted enterprise

A payment operator runs Batteries and the signer for several enterprises. Each enterprise has its own wholesale allocation, labelled with its `enterprise_id`, in the operator's database. Grant, allocation, key, escrow, webhook token, and the export topic stay with that operator. The enterprise receives the allocation key once and stores it in its vault. It consumes usage events whose `subject` is its `enterprise_id`. It does not open the ledger.

## Remaining gaps in provisioning and funding

Programmatic create, fund, and revoke exist on the Batteries management API. The [management integration draft](batteries-management-integration.md) maps them to the engine's `PaymentProvider`. Cloud SPE does not build that server. Caller idempotency, per-allocation balance, and attributed usage were delivered on Batteries `main` at `501c1ed`. These behaviours remain open:

- The CLI still posts a retried fund twice. Management API retries are safe only when the engine resends the same `Idempotency-Key` and the same fields. Keys are shared by every caller of the grant, so the engine namespaces its own.
- Management credentials are global and loaded from a static file, so a tenant cannot be given its own scoped credential.
- `enterprise_id` can be set only when an allocation is created. Changing it means a new allocation and key, a vault key change, and ended sessions.

The first Cloud SPE client of that API stores the key in the vault entry for `sub`. A lost create response can be replayed, including the secret. It reads remaining balance from `GET /v1/allocations/{id}` (`available_usd` or `available_eth`). Per-job cost is pulled through the [cost-sync adapter](../references/analysis/2026-10-06-Batteries-Cost-Sync-Adapter-Interface.md). The [export topic](usage-event-export.md) stays an optional push path.

A later design may add a payment-processor integration: the enterprise pays through a processor, and a service turns confirmed payments into allocation funding through that same admin API.

## Retail boundary

Product plans, checkout, invoices, customer credit, and markup are enterprise features. The reference application may mock them against the usage export. Sponsoring an allocation does not implement those features. Network fee, on-chain settlement, and the customer charge remain three different figures. The export carries the network fee. The enterprise applies its own price.

## Decisions

- The webhook and the allocation key stay between the signer and Clearinghouse in every mode.
- Batteries is the wholesale network-credit ledger and signing authorizer. It stores no end-user records.
- A grant is the customer budget. An allocation is a partition of that grant. Several keys can hang off one allocation.
- Enterprise deployments start with one wholesale grant per enterprise, labelled with `enterprise_id`, with allocations provisioned through the management API.
- The enterprise application owns end users, per-user limits, attribution, and billing. It stores the allocation key in a user-scoped vault on the enterprise app's authentication server.
- Standalone and self-operated provisioning is the management API or the CLI. Hosted provisioning is a key delivered once plus the export stream.
- Programmatic allocation create and fund use the management API, with grant-scoped `Idempotency-Key` retries. Allowance comes from the allocation read. Per-job cost comes from the usage pull.

## Work this design implies

`netspe-cz5.6` is the vault resolver, seeded by the management API or the CLI. `netspe-cz5.1` is closed: the management API is on Batteries `main`. `netspe-cz5.16` is the first Cloud SPE client of that API. `netspe-scr.12`, `netspe-scr.13` and `netspe-scr.14` are closed: the maintainer delivered those three asks at `501c1ed`.
