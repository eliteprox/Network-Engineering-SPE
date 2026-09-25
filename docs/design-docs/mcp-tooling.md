# MCP Tooling

**Status:** Draft for review
**Updated:** 25 September 2026
**Context:** [Gateway server routes](gateway-server-routes.md), [capability matrix](console-capability-and-gap-matrix.md)

MCP and REST are two adapters over the gateway core. Tool behavior matches the routes in the gateway draft. Console at `009a703d7b6434bab905902375f562e5980728af` is the behavior reference for which tools exist. It is not the runtime host.

No Cloud SPE decision record accepts this contract yet.

## Core tools

These tools belong to the shared journey. Each one calls the same core function as the matching HTTP route.

| Tool | HTTP counterpart | Behavior |
| --- | --- | --- |
| `list_capabilities` | `GET /v1/capabilities` | Advertised capabilities and freshness |
| `describe_capability` | `GET /v1/capabilities/lookup?name=` | Mode, inputs, and schema hints for the exact advertised name |
| `get_pricing` | `GET /v1/capabilities/rate?name=` | Network rate, unit, and source. Optional `price_usd` when [go-livepeer#4095](https://github.com/livepeer/go-livepeer/pull/4095) is present |
| `run_capability` | `POST /v1/jobs` | Persist, then invoke. Progress notifications while the job runs. The tool returns the same payload as the job status route |
| `get_cost_report` | `GET /v1/usage` | Owner-scoped network-cost projection, with pending and unmatched rows visible |
| `me` | `GET /v1/me` | Opaque actor, scope, and public client identity from the access adapter |

`describe_capability` and `get_pricing` take the advertised name as a tool argument, including names of the form `pipeline/app-name`. The slash never becomes a URL path segment inside the MCP adapter. The adapter passes `name` as a query parameter to the core.

`run_capability` sends MCP progress notifications (`notifications/progress`) for elapsed wait and queue status. A progress notification reports that work is still running. It is not a media frame, a token of model output, or proof that the job succeeded. The terminal result is the tool return value or a later read of the job.

## Extension tools

Console also registers `upload`, `upload_image`, `create_upload_url`, `get_recent_assets`, `search_assets`, and `forget_assets`. Those tools stay in the enterprise or reference application. Core documents a public HTTPS URL as the input baseline. An extension may add tools on an enterprise MCP host. It uses the core through the public package interface or the HTTP API, and it does not fork the core tool list.

`get_cost_report` and `me_usage` in the pinned Console server read PymtHouse and OpenMeter for the current UTC day. The core tools read the engine projection instead. Retail spend and `hasAccess` limits stay in the enterprise.

## Persistent sessions

The gateway server may expose streaming endpoints for persistent Live Runner sessions. Those endpoints are HTTP routes on `livepeer-python-gateway-server`. They replace `@pymthouse/gateway-web` for that traffic. An MCP tool may start a session and return a handle. The media stream itself is the streaming route, authenticated with the same access adapter as the JSON API.

This draft leaves open whether any MCP client can consume that stream through the tool protocol. Until a pinned client demonstrates it, persistent media is an HTTP route and `run_capability` covers jobs that complete.

## Access

The MCP resource publishes protected-resource metadata as described in the [authorization draft](enterprise-authorization-server.md). Standalone deployments accept an operator-issued bearer API key. Enterprise deployments accept an access token from the configured authorization server. Both produce the same internal actor context before a tool runs.

## Decisions

- Six core tools share the gateway route implementations.
- Asset and upload tools are extensions.
- Progress notifications and media streams are different transports.
- Capability names with a slash are tool arguments, then query parameters.

## Work this design implies

`netspe-cz5.8` is the six core tools tied to the route handlers. `netspe-cz5.10` is the streaming-versus-MCP spike with a named client and a named persistent capability.
