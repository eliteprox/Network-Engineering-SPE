# Design Document Index

Design documents capture durable constraints and cross-system choices for Cloud
SPE deliverables. A design is authoritative only within that scope, when its
status is `Accepted` and it links to the decision that approved it.

## Current documents

| Document | Status | Purpose |
| --- | --- | --- |
| [Self-sovereign open builder stack](self-sovereign-open-builder-stack-draft.md) | Primary working proposal | Executive summary, component/repository ownership, stakeholder alignment, technical scope and acceptance |
| [Builder engine diagrams and sequences](open-builder-architecture-and-sequences.md) | Companion to primary proposal | Components, enterprise integration modes, payment operation, execution and accounting |
| [Capabilities and gap matrix](console-capability-and-gap-matrix.md) | Supporting pinned evidence | Current implementations, new homes and verification gaps |
| [Core beliefs](core-beliefs.md) | Proposed | Agent-first and Cloud SPE delivery principles |
| [September–December milestone proposal](cloud-spe-september-december-2026-milestones-draft.md) | Historical draft awaiting reviewed scope | Input to later October–December milestone revision |
| [Gateway server routes](gateway-server-routes.md) | Draft for review | Live Runner HTTP surface, slash-safe capability names, streaming routes |
| [MCP tooling](mcp-tooling.md) | Draft for review | Core tools shared with the gateway routes; extension tools and media streams |
| [Enterprise authorization server](enterprise-authorization-server.md) | Draft for review | Pluggable issuer, three credentials, and the invocation sequence |
| [Usage event export](usage-event-export.md) | Draft for review | CloudEvents export topic and OpenMeter, Lago, and Kill Bill connectors |
| [Clearinghouse configuration](clearinghouse-configuration.md) | Draft for review | Current serve flags and the optional usage export topic |
| [Payment provisioning modes](payment-provisioning-modes.md) | Draft for review | Wholesale allocation per enterprise, CLI and hosted provisioning, open funding gap |

Intermediate Console-replacement and enterprise deployment drafts were removed
on 23 September after consolidation. Original stakeholder evidence and its chronology remain in the
[reference catalog](../references/index.md); the primary proposal presents the
current direction.
The six integration drafts dated 25 September 2026 are review material for the
gateway, MCP, authorization, usage export, Clearinghouse flags, and
provisioning. A feature roadmap is derived from them after review. No draft
becomes accepted by being linked. Work state remains in Beads.
