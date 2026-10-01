# Design Document Index

Design documents capture durable constraints and cross-system choices for Mike
Zupper's Build Track deliverables. A design is authoritative only within that scope, when its
status is `Accepted` and it links to the decision that approved it.

## Current documents

| Document | Status | Purpose |
| --- | --- | --- |
| [Self-sovereign open builder stack](self-sovereign-open-builder-stack-draft.md) | Accepted; effective 30 September | Executive summary, component/repository ownership, stakeholder alignment, technical scope and acceptance |
| [Builder engine diagrams and sequences](open-builder-architecture-and-sequences.md) | Accepted architecture companion | Components, enterprise integration modes, payment operation, execution and accounting |
| [Capabilities and gap matrix](console-capability-and-gap-matrix.md) | Supporting pinned evidence | Current implementations, new homes and verification gaps |
| [Build Track December delivery task breakdown](build-track-december-2026-task-breakdown-draft.md) | Accepted delivery plan; owner Mike Zupper | Six September planning tasks plus 31 implementation tasks across five milestones, work kinds, blocking dependencies, and separate stretch goals; implementation assignees and task-level dates pending |
| [Build Track December delivery issues](build-track-december-2026-issue-proposal-draft.md) | Accepted delivery mapping; owner Mike Zupper | 45 tasks (six planning and 39 implementation) with milestone, home, acceptance, dependencies, and source-task traceability; six planning repository issues created; implementation homes, assignments, and task-level dates pending |
| [Core beliefs](core-beliefs.md) | Proposed | Agent-first and Cloud SPE delivery principles |
| [September–December milestone proposal](cloud-spe-september-december-2026-milestones-draft.md) | Historical; superseded by accepted M1–M5 plan | Input to later October–December milestone revision |
| [Gateway server routes](gateway-server-routes.md) | Draft for review | Live Runner HTTP surface, slash-safe capability names, streaming routes |
| [MCP tooling](mcp-tooling.md) | Draft for review | Core tools shared with the gateway routes; extension tools and media streams |
| [Enterprise authorization server](enterprise-authorization-server.md) | Draft for review | Pluggable issuer, public MCP OAuth, and a user-scoped vault for the allocation key |
| [Usage event export](usage-event-export.md) | Draft for review | CloudEvents export topic and OpenMeter, Lago, and Kill Bill connectors |
| [Clearinghouse configuration](clearinghouse-configuration.md) | Draft for review | Current serve flags and the optional usage export topic |
| [Payment provisioning modes](payment-provisioning-modes.md) | Draft for review | Wholesale grant per enterprise, vault-held allocation key, remaining provisioning gaps |
| [Batteries management integration](batteries-management-integration.md) | Draft for review | `PaymentProvider` over the Batteries management API: single-tenant reseller, upstream asks, engine-side multi-tenancy |
| [Enterprise authentication provider modes](enterprise-auth-provider-modes.md) | Draft for review | Provider chain; Basic, OIDC, MCP PKCE, CIMD catalog, device flow, audience token exchange and local gateway |
| [simple-infra migration](simple-infra-builder-migration.md) | Draft for review | Second-application evidence register, seam and route map, phased move onto the engine |

Intermediate Console-replacement and enterprise deployment drafts were removed
on 23 September after consolidation. Original stakeholder evidence and its chronology remain in the
[reference catalog](../references/index.md); the accepted architecture presents the current direction.
The [approval record](../decisions/2026-09-30-build-track-architecture-and-milestones.md)
and [specification](../product-specs/build-track-2026.md) define the accepted baseline.
Historical `-draft` filenames remain for link stability. GitHub Project 13 tracks
public milestones; Beads tracks internal work.
The six integration drafts dated 25 September 2026 are review material for the
gateway, MCP, authorization, usage export, Clearinghouse flags, and
provisioning. Feature and story order lives in Beads under `netspe-cz5`.
The three drafts dated 1 October 2026 test the accepted engine against its
neighbours: Batteries management, authentication modes, and the simple-infra
migration. Their stories live under `netspe-scr`. No draft
becomes accepted by being linked. Work state remains in Beads.
