# Payment Provisioning Modes

**Status:** Draft for review
**Updated:** 25 September 2026
**Context:** [Enterprise authorization server](enterprise-authorization-server.md), [usage event export](usage-event-export.md), [architecture companion](open-builder-architecture-and-sequences.md#payment-operation-choices)

This draft maps how much of the Clearinghouse grant model a deployment exposes. It uses the payment modes from the architecture companion and the access modes from the [authorization draft](enterprise-authorization-server.md). Those choices are independent: an enterprise authorization server can sit on a self-operated ledger or on a hosted payment operator.

No Cloud SPE decision record accepts this contract yet.

## What the webhook already does

`POST /v1/signer/authorize` is a signer-to-Clearinghouse call. The signer sends `Livepeer-Clearinghouse-Token`. The body carries the gateway's allocation key as `Authorization: Bearer`. A matching active key with an active grant, an allocation in `active` status, a positive available balance, and a coherent session binding returns HTTP 200 and JSON `status` 200 plus `auth_id`. The session binds state id, app, payment type, orchestrator, and key.

The gateway participates by configuring that allocation key on the remote signer headers. Public job routes do not create keys. Revocation stops later authorizations. Rotating or revoking the key ends payment sessions bound to it; the gateway must load the replacement key. Tickets already signed can still arrive and post. Creating or funding an allocation does not fund signer escrow. Escrow funding stays an operator action on the signer wallet.

Grant, allocation, and key operations today are CLI commands. Fund operations are not caller-idempotent: each call inserts a new `fund:` ledger key.

## Wholesale accounting model

Batteries has no user, customer, or account record. Its records are grants and allocations (budgets), API keys (gateway credentials), payment sessions (one per signer payment state), and signed-ticket usage events. It keeps an event-level audit trail, but that trail rolls up to the allocation balance. Record volume grows with traffic, not with the number of end users.

Enterprise deployments therefore use one wholesale network-credit allocation per enterprise. Batteries authorizes signing while that allocation has a positive available balance, debits each signed ticket, and reports network cost as expected value in wei. It does not know which end user caused a cost.

The enterprise application owns everything below that allocation:

- end-user identities, allowances, subscriptions, and per-user spend limits
- retail rating, pricing, markup, invoices, and payment collection
- per-user or per-job attribution, joined from its own job records to the [usage export](usage-event-export.md)

Because every end user shares the allocation, Clearinghouse cannot stop one user from exhausting the enterprise's budget. It returns decision status 402 only when the whole allocation is exhausted. The gateway or enterprise application enforces per-user limits, including during metered sessions and not only before invocation. A separate allocation per end customer remains possible later, but it would still be a budget label rather than a user record.

## Standalone

The operator uses the CLI: one grant, one allocation, one API key, installed in the gateway. Callers authenticate to the gateway with an operator-issued gateway API key, or anonymously only on a local test profile that is not a public deployment. Callers never see grant ids. The engine reports network cost for that single allocation. `enterprise_id` metadata may be omitted when there is one tenant.

## Self-operated enterprise

The enterprise operates its own Batteries and signer. One wholesale allocation per enterprise is provisioned with the CLI on the Batteries host, with `enterprise_id` set in allocation metadata at creation for the [usage export](usage-event-export.md). The operator installs the allocation key in the gateway. End-user routes and MCP tools do not create, fund, or read allocations.

## Hosted enterprise

A payment operator runs Batteries and the signer for several enterprises. Each enterprise has its own wholesale allocation, labelled with its `enterprise_id`, in the operator's database. Grant, allocation, key, escrow, webhook token, and the export topic stay with that operator. The enterprise receives a service credential and consumes usage events whose `subject` is its `enterprise_id`. It does not open the ledger.

## Open gap: allocation provisioning and funding

This set of drafts does not define how an enterprise's allocation is created and funded when the enterprise application pays for network usage. Today an operator runs CLI commands on the Batteries host. That leaves these behaviours unaddressed:

- Remote or programmatic provisioning requires host access, which is broader than a provisioning role.
- A retried fund posts twice. The grant's unallocated balance caps the damage, but only revoking the allocation returns funds, and revoking ends its sessions.
- `enterprise_id` can be set only when an allocation is created. Changing it means a new allocation and key, a gateway key change, and ended sessions.
- Enterprise systems have no programmatic read of the remaining allocation balance. The export reports spend, not balance.

A later design may add a payment-processor integration: the enterprise pays through a processor, and a service turns confirmed payments into allocation funding. That design would own operator or service authentication, caller-idempotent funding, allocation metadata updates, balance reads, and hosted-operator boundaries. It corresponds to requirement A3 in the [capability matrix](console-capability-and-gap-matrix.md) ("authenticated, retry-safe provisioning"), which remains open. An engine- or Batteries-hosted admin adapter was considered for this role and is withdrawn from the current drafts.

## Retail boundary

Product plans, checkout, invoices, customer credit, and markup are enterprise features. The reference application may mock them against the usage export. Sponsoring an allocation does not implement those features. Network fee, on-chain settlement, and the customer charge remain three different figures. The export carries the network fee. The enterprise applies its own price.

## Decisions

- The webhook and the allocation key stay between the signer and Clearinghouse in every mode.
- Batteries is the wholesale network-credit ledger and signing authorizer. It stores no end-user records.
- Enterprise deployments use one wholesale allocation per enterprise, labelled with `enterprise_id`.
- The enterprise application owns end users, per-user limits, attribution, and billing.
- Standalone and self-operated provisioning is the CLI. Hosted provisioning is a service credential plus the export stream.
- Programmatic allocation provisioning and funding is an open gap for a later design.

## Work this design implies

A later roadmap bead should split CLI-only standalone and self-operated setup from hosted credential configuration. Acceptance for CLI-only setup is one grant, one allocation with `enterprise_id` metadata, and one key installed in the gateway, with an end-user token unable to reach any allocation operation. Allocation provisioning and funding by a payment-processor integration is tracked separately as later design work.
