# Gateway Server Routes

**Status:** Draft for review
**Updated:** 25 September 2026
**Context:** [Self-sovereign open builder stack](self-sovereign-open-builder-stack-draft.md), [capability matrix](console-capability-and-gap-matrix.md)

This draft specifies the HTTP surface of a proposed `livepeer-python-gateway-server` package built on `livepeer/livepeer-python-gateway`. REST and MCP call the same core. The package holds the remote-signer URL and discovery URL. Caller credentials stop at the access adapter described in the [authorization draft](enterprise-authorization-server.md). The Clearinghouse allocation API key is never a field on public requests. Enterprise mode loads it from the vault on the enterprise app's authentication server after the access token is verified. Standalone mode keeps it in server configuration.

No Cloud SPE decision record accepts this contract yet.

## Execution path

Job invocation uses the Live Runner interfaces in `livepeer-python-gateway`, including `call_runner` and the live session types. The server is the process boundary that replaces the Console prototype's hosted gateway package. Discovery rates come from the signer and orchestrator evidence the SDK already reads.

[go-livepeer#4095](https://github.com/livepeer/go-livepeer/pull/4095) is open. It adds `price_usd` to remote discovery. Rate responses should carry that field when the signer revision includes it, and should omit it when the field is absent. A discovery rate is a published unit price with its source and freshness. It is a binding quote for a future job only when a later decision says so.

## Capability names

Orchestrators advertise capabilities as `pipeline/app-name`. That slash is part of the name. A path parameter such as `/v1/capabilities/{name}` splits the name across segments. Percent-encoding the slash as `%2F` is unsafe: many proxies and routers decode it before routing, so the request never matches the intended handler.

The advertised name is a query parameter. Job identifiers are opaque and may stay in the path.

| Method and path | Role |
| --- | --- |
| `GET /v1/capabilities` | List advertised capabilities, modes, and freshness |
| `GET /v1/capabilities/lookup?name=pipeline/app-name` | One capability: inputs, mode (`single-shot` or `persistent`), and schema hints |
| `GET /v1/capabilities/rate?name=pipeline/app-name` | Network rate, unit, source, and optional `price_usd` |
| `POST /v1/jobs` | Persist a job, then invoke. The body carries a caller idempotency key |
| `GET /v1/jobs/{id}` | Status, result reference, or recoverable handle |
| `GET /v1/usage` | Owner-scoped network-cost projection, including pending and unmatched rows |
| `GET /v1/me` | Opaque actor already validated by the access adapter |

`POST /v1/jobs` returns a job id as soon as the runner accepts queued work. The caller reads `GET /v1/jobs/{id}` for completion. Repeating the same idempotency key returns the original job and does not submit a second run.

## Streaming routes

Persistent Live Runner sessions use trickle or websocket media. Those sessions are dedicated streaming routes on this server, one route per supported execution mode, with the same access adapter as the JSON routes. The route set is fixed when a mode is accepted. MCP progress notifications cover jobs that finish. They do not carry the media stream. Whether a persistent session can be represented as an MCP tool remains open until one streaming route is implemented and exercised with a pinned client.

## Boundaries

Upload storage, asset libraries, retail prices, invoices, grant administration, and the signer authorization webhook are outside this surface. Provisioning lives in the [provisioning draft](payment-provisioning-modes.md) and is not part of these routes. Usage rows on `GET /v1/usage` are the engine projection defined in the [usage export draft](usage-event-export.md).

The Daydream SDK service can select a signer and a discovery URL from a per-key validate response. This server uses the signer URL and discovery URL from its own configuration. The allocation key comes from the vault in enterprise mode and from server configuration in standalone mode. Per-key signer selection is a hosted-product behavior and stays out of this contract.

## Decisions

- Capability identity in URLs is the query parameter `name`, because advertised names contain a slash.
- Invocation is Live Runner through the Python SDK. The server package is the process boundary for Live Runner HTTP.
- Persistent media has its own HTTP routes. Job polling stays on `GET /v1/jobs/{id}`.
- The allocation API key is not a field on public requests. Enterprise mode resolves it from the vault on the enterprise app's authentication server. Standalone mode keeps it in server configuration.

## Work this design implies

`netspe-cz5.7` is the JSON routes, the idempotent job record, the query-parameter capability lookup, and one proxied Live Runner job. `netspe-cz5.9` is one streaming route. `GET /v1/usage` may show pending rows joined on `manifest_id` before the export topic exists.
