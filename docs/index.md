# Documentation Index

This index is the entry point for knowledge about the Cloud SPE's assigned
portion of the wider Network Engineering SPE. It is not a programme-wide
catalog. Cloud SPE work status and dependency ordering live in Beads (`bd ready`,
`bd blocked`), not here.

## Durable guidance

- [Accepted delivery specification](product-specs/build-track-2026.md) — architecture, scope, milestone gates and tracking convention.
- [September 30 acceptance decision](decisions/2026-09-30-build-track-architecture-and-milestones.md) — review outcome and approval basis.
- [Public milestone project](https://github.com/orgs/Cloud-SPE/projects/13) — ecosystem transparency; Beads remains Mike's internal tracker.

- [Accepted builder-engine architecture](design-docs/self-sovereign-open-builder-stack-draft.md)
  — stakeholder executive summary, component/repository ownership map, and
  accepted baseline effective 30 September, following stakeholder review.
- [Architecture diagrams and sequences](design-docs/open-builder-architecture-and-sequences.md)
  — repository roles, imported/service integration, self-operated/hosted payments,
  shared execution and separate accounting.
- [Capability and gap inventory](design-docs/console-capability-and-gap-matrix.md)
  — pinned implementation evidence, preserved Console/Batteries source map,
  MCP behavior, new homes and unverified integration gaps.
- [Build Track December delivery task breakdown](design-docs/build-track-december-2026-task-breakdown-draft.md)
  — Mike Zupper's accepted plan of six September planning tasks plus 31 implementation tasks across five milestones,
  with completion evidence, upstream dependencies, and separate stretch goals;
  tracked in the public project, with implementation assignments refined during execution.
- [Build Track December delivery issues](design-docs/build-track-december-2026-issue-proposal-draft.md)
  — Mike Zupper's 45 delivery tasks (six planning and 39 implementation) with single milestones, proposed repository homes,
  acceptance criteria, dependencies, and a crosswalk to the implementation source tasks;
  mirrored as project drafts, with upstream handoffs kept separate.
- [Integration contract drafts](design-docs/index.md)
  — gateway routes, MCP tools, enterprise authorization, usage export,
  Clearinghouse flags, and provisioning modes, listed in the design index.
- [Repository architecture](../ARCHITECTURE.md) — boundaries, information model,
  and source precedence.
- [Core beliefs](design-docs/core-beliefs.md) — principles for shaping the SPE
  and its agent-facing environment.
- [Draft September–December 2026 Cloud SPE milestones](design-docs/cloud-spe-september-december-2026-milestones-draft.md)
  — historical August planning input, superseded by the accepted M1–M5 plan. Demand generation and adoption remain excluded.
- [Design document index](design-docs/index.md) — accepted and proposed designs.
- [Product specification index](product-specs/index.md) — intended outcomes and
  acceptance contracts.
- [Decision index](decisions/index.md) — accepted decisions and open decision
  records.
- [Quality review](QUALITY.md) — current repository and Cloud SPE delivery-readiness gaps.

## Monthly updates

- [September 2026 Build Track monthly update](updates/2026-09-build-track-monthly-update.md) — draft for Mike’s review; not yet published to GitHub status updates or the Livepeer forum.

## Source material and evidence

- [October 1 builder-layer proposal](references/analysis/2026-10-01-Builder-Layer-Abstraction-and-Two-Application-Review.md) and [separate assessment](references/analysis/2026-10-01-Builder-Layer-Proposal-Assessment.md) — subsequent M1 supporting analysis and concrete M2 contract-review input; accepted scope remains unchanged.

- [John–Josh auth and API-key discussion capture](references/stakeholder-input/meetings/2026-09-26-John-Josh-Auth-and-API-Key-Discussion.md)
  — supplied conversation and separate assistant assessment, recorded 26 September;
  reference only, with no change to the planning baseline.

- [24 September Agent–Network Engineering SPE sync findings](references/stakeholder-input/meetings/2026-09-24-Agent-NE-SPE-Sync-Findings.md)
  — conceptual alignment, source handoffs, Console clarification and hosting
  boundaries; links the preserved transcript and dates the simple-infra update.

- [23 September John–Mike architecture discussion](references/stakeholder-input/meetings/john-mike-build-track-architecture-discussion-0923-2026.txt)
  — supplied transcript; working alignment on the reusable backend, enterprise
  boundaries and upstream scope, qualified in the primary architecture.

- [21 September Mike–Josh clearinghouse-batteries conversation](references/stakeholder-input/meetings/2026-09-21-Mike-Josh-Clearinghouse-Batteries-Conversation.md)
  — supplied conversation, Inc's stated support direction, payment-core scope,
  credential options, ticket-EV accounting, and the seven-outcome boundary.
- [21 September capture of Josh's Payments Clearinghouse proposal](references/source-material/2026-09-21-Josh-Payments-Clearinghouse-Reference.md)
  — preserved Notion PDF, provenance, source limitations, and payment-core
  implications for the proposed shared open-source clearinghouse.
- [Complete chronological reference index](references/index.md) — canonical
  catalog of every versioned reference, including its date, exact context,
  functional category, lifecycle status, participants, and provenance limits.
- [Build Track outcome and concepts](references/analysis/2026-08-27-Build-Track-Outcome-and-High-Level-Concepts.md)
- [Build Track repository traceability](references/analysis/2026-08-24-Build-Track-Repo-Traceability.md)
- [Network Engineering SPE II notes](references/source-material/2026-08-25-NetworkEngieneerSPE2-Notes-v2.md)
- [21 August Build Track alignment transcript extract](references/source-material/2026-08-21-Build-Track-Alignment-Transcript-Extract.md)
  — timestamped historical evidence relevant to the Cloud SPE scope.
- [2 September John Mull clearinghouse meeting notes](references/stakeholder-input/meetings/2026-09-02-John-Mull-Clearinghouse-Meeting-Notes.md)
  — reported current-state findings, outcome gaps, and evidence required for a
  follow-up with Elite Encoder.
- [2 September Rich, Doug, and Hunter Build Track feedback](references/stakeholder-input/meetings/2026-09-02-Rich-Doug-Hunter-Build-Track-Feedback.md)
  — stakeholder input on self-sovereign payment, expected performance, failure
  recovery, and recourse; recorded as unresolved direction rather than an
  approved requirement.
- [8 September Rick Build Track discussion consolidated notes](references/stakeholder-input/meetings/2026-09-08-Rick-Build-Track-Survey-Discussion-Consolidated-Notes.md)
  — evidence-qualified synthesis of the proposed minimal builder architecture,
  Open Clearinghouse, provisional December deliverables, Cloud SPE boundary,
  and decisions still requiring external or joint approval.
- [9 September Doug Agent GTM and Build Track alignment meeting context](references/facilitation/meeting-guides/2026-09-09-Doug-Agent-GTM-Build-Track-Alignment-Meeting-Context.md)
  — request, purpose, scope boundaries, and intended evidence for the
  30-minute alignment discussion.
- [9 September Doug Agent GTM and Build Track alignment findings](references/stakeholder-input/meetings/2026-09-09-Doug-Agent-GTM-Build-Track-Alignment-Meeting-Findings.md)
  — evidence-qualified synthesis of the three access paths, commercial Agent
  boundary, open Agent handoff, minimal clearinghouse direction, security risk,
  provisional December outcome, and decisions still requiring approval.
- [9 September emerging Build Track architecture](references/analysis/2026-09-09-Build-Track-Emerging-Architecture.md)
  — standalone architecture image and evidence-qualified explanation of the
  three access paths, shared builder layer, discovery and payment control,
  Live Runner execution, responsibility boundaries, and unresolved decisions.

## Historical architecture alignment materials

The survey/workshop sequence below is preserved as facilitation material.
Current work follows proposal-led review; these guides are not mandatory
prerequisites to reviewing the architecture.

- [Survey and workshop process](references/facilitation/2026-08-26-Build-Track-Architecture-Alignment-Process.md)
  — scheduling sequence, roles, decision classifications, and required outputs.
- [Survey administration](references/facilitation/surveys/2026-08-27-Build-Track-Architecture-Survey.md) —
  owner guidance for distribution, private collection, and synthesis.
- [Survey instructions](references/facilitation/surveys/2026-08-27-Build-Track-Architecture-Survey-Instructions.md)
  — respondent guidance and neutral agent-interview prompt.
- [Survey template](references/facilitation/surveys/2026-08-27-Build-Track-Architecture-Survey-Template.md) —
  the fillable 10–15 minute Markdown questionnaire.
- [Workshop Part 1](references/facilitation/workshops/2026-08-26-Build-Track-Architecture-Workshop-Part-1.md) — a
  60-minute outcomes, current-state, and target-architecture facilitator guide.
- [Workshop Part 2](references/facilitation/workshops/2026-08-26-Build-Track-Architecture-Workshop-Part-2.md) — a
  60-minute architecture decision, ownership, and requirements facilitator
  guide.
- [Doug Build Track feedback discussion guide](references/facilitation/meeting-guides/2026-09-02-Doug-Build-Track-Feedback-Discussion-Guide.md)
  — a 60-minute 15/30/15 agenda for confirming the August vision, architecture
  implications, milestone intent, and decision authority.
- [John Mull clearinghouse roadmap discussion guide](references/facilitation/meeting-guides/2026-09-02-John-Mull-Clearinghouse-Roadmap-Discussion-Guide.md)
  — a 60-minute 15/30/15 fact-finding agenda covering Pymthouse provenance,
  deployed behavior, contract coverage, ownership, and roadmap.
- [8 September Rick survey discussion agenda](references/facilitation/meeting-guides/2026-09-08-Rick-Build-Track-Survey-Discussion-Agenda.md)
  — a 60-minute decision-oriented agenda covering the December deliverable
  hypothesis, self-sovereign Agent ambiguity, SDK-versus-gateway architecture,
  discovery, payment, service assurance, ownership, and acceptance evidence.

Files in `references/` are not automatically normative. Their status, date,
method, and verification limits determine how they may be used.
