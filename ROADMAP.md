# Musan roadmap

Status: draft for discussion. No implementation has started. Dates and effort estimates are intentionally unset until scope, runtime mode, and first integrations are chosen.

This document owns phase scope, sequencing, deliverables, dependencies, and completion gates. The [PRD](docs/PRD.md) owns product behavior, requirement IDs, and acceptance criteria. The [research](docs/research/api-access-landscape.md) provides evidence for product and dependency choices.

Repository workflow and agent instructions live in [AGENTS.md](AGENTS.md).

Only the inclusion of API access management, webhooks, and MCP in phase 1 is confirmed. The milestones below are proposed. Later work is conditional on adopter needs.

## Phase overview

| Phase | Outcome | Status |
| --- | --- | --- |
| 0 — Requirements and design | Agreed scope and contracts sufficient to start a bounded implementation. | PRD and research drafted; decisions and design open. |
| 1 — First usable release | Application API access, outgoing webhooks, and protected MCP tools work together through the chosen integrations. | Not started. |
| 2 — Adoption-driven expansion | Broader ecosystem coverage and selected improvements supported by adopter evidence. | Candidate work; not committed. |

## Phase 0 — Requirements and design

### Deliverables

- PRD, software comparison, and this roadmap.
- Decisions on embedded versus service-backed operation, webhook meaning, MCP meaning, and first languages/frameworks.
- Capability-by-capability build-versus-reuse decisions, including deployment and license constraints.
- Architecture/design document covering trusted identity, tenant boundaries, permission evaluation, credential lifecycle, shared limits, event durability, and worker ownership.
- Proposed contracts, compatibility matrix, measurable operating targets, and a scoped implementation backlog.

### Completion gate

Resolve the decisions in [PRD section 13](docs/PRD.md#13-decisions-needed-before-implementation) that affect the initial implementation. Select one initial runtime mode, unless evidence justifies both. Choose the first two language integrations, OAuth provider/client matrix, and webhook delivery approach. Define revocation, retry, retention, concurrency, and latency targets sufficiently to test them.

This phase remains documentation-only. Finishing a design does not itself authorize implementation.

## Phase 1 — First usable release

### Scope

| Capability | Release boundary |
| --- | --- |
| API access | Credential issuance, expiration, revocation and rotation; tenant/environment isolation; named permissions; rate and usage admission. |
| Webhooks | Authorized outgoing subscriptions, signed events, durable acceptance, retries, inspection, and controlled replay. |
| MCP | Explicitly registered application tools over protected remote HTTP, using an existing compatible OAuth authorization server. |
| Integration | Framework-neutral contracts, two selected language integrations, and representative host examples. |
| Management | Host-usable management operations, with administrator authority separated from consumer access. |
| Operations | Redacted diagnostics, auditability, documented consistency and outage behavior, and native package distribution. |

Outgoing webhooks and application MCP tools remain working interpretations until phase 0 resolves them. If those interpretations change, revise this scope and the PRD together.

### Milestone 1.1 — Shared contracts and API access

**Deliver:** shared application/tenant/environment/principal/permission vocabulary; decision and error contracts; trusted host identity integration; API-key lifecycle; resource ownership checks; limits; management operations; conformance cases; and initial language bindings. Establish the Turborepo monorepo with native toolchain and packaging boundaries during implementation.

**Depends on:** phase 0 runtime, language, ownership, and dependency decisions.

**Gate:** representative allowed and denied requests behave consistently across the selected integrations. Rotation, revocation, concurrent admission, tenant isolation, and dependency outages meet the agreed guarantees.

**PRD references:** INT-01–INT-05, KEY-01–KEY-03, AUTH-01–AUTH-03, LIM-01–LIM-04, and section 9 monorepo requirements.

### Milestone 1.2 — Outgoing webhooks

**Deliver:** event registration, authorized subscriptions, endpoint lifecycle, per-endpoint signing and secret rotation, durable acceptance, retry/exhaustion behavior, redacted inspection, and replay.

**Depends on:** shared access contracts and the selected persistence/delivery approach. Event and subscription contracts can be designed alongside milestone 1.1.

**Gate:** pending accepted deliveries survive worker restart; duplicate delivery is documented and identifiable; failed attempts follow the retry policy; revoked subscriptions cannot dispatch pending unauthorized events or replay them; destination restrictions and signature verification pass acceptance cases.

**PRD references:** WH-01–WH-09.

### Milestone 1.3 — Protected MCP tools

**Deliver:** explicit tool registration, schemas and permission mappings; OAuth resource-server integration; authorization-dependent discovery; per-invocation resource checks; shared admission limits; and redacted tool audit records.

**Depends on:** shared access contracts and the selected MCP version, authorization provider, and independent client matrix. This milestone can proceed alongside webhook integration once those foundations exist.

**Gate:** selected clients complete authorization; invalid-audience tokens fail; directly calling a hidden or unauthorized tool never executes its handler; extra OAuth scopes cannot bypass application grants; API and MCP use the shared budget when configured.

**PRD references:** MCP-01–MCP-07.

### Milestone 1.4 — Cross-language and operational readiness

**Deliver:** consuming-application examples, supported-runtime matrix, native package release preparation, operator guidance, safe diagnostics, and acceptance evidence across the three interfaces.

**Depends on:** the API, webhook, and MCP milestones meeting their individual gates.

**Release gate:** all phase-1 Must requirements and applicable nonfunctional requirements pass. Demonstrate the [PRD acceptance scenarios](docs/PRD.md#11-product-acceptance-scenarios), including equivalent access decisions through two language integrations. Record measured behavior and limitations; resolve isolation, unauthorized execution, or secret-leakage failures before release.

Milestone numbers describe dependency order, not separate product phases. **Webhooks and MCP remain part of phase 1.** API access alone does not complete the first release.

### Deferred from phase 1

- Universal language/framework coverage or a second runtime mode without adopter evidence.
- A full standalone dashboard, developer portal, gateway, or identity provider.
- A custom OAuth authorization server or a new general-purpose policy language.
- Inbound third-party webhook routing or general workflow orchestration.
- Automatically publishing every route as an MCP tool, or MCP tools for administering Musan.
- Billing, financial-grade metering, a global hosted platform, or multi-region guarantees.

PRD Should requirements are candidates, not first-release blockers: INT-06, AUTH-04, LIM-05, WH-10, and MCP-08.

## Phase 2 — Adoption-driven expansion

Choose work using pilot feedback and the [PRD success measures](docs/PRD.md#12-success-measures-and-validation). Candidate work includes:

- Additional language SDKs, framework adapters, and consuming-tool integrations.
- Permission bundles, weighted usage costs, and richer usage diagnostics.
- Test-event delivery and failing-endpoint lifecycle notifications.
- Local stdio MCP support where adopters need it.
- Embeddable management components or a reference UI.

For each selected item, record the adopter need, update or add PRD requirements, define dependencies and acceptance criteria, and then schedule a milestone. A second deployment mode, richer policy backend, inbound webhooks, or hosted offering requires its own scope decision; none is promised by this roadmap.

**Completion gate:** each selected capability passes its documented acceptance criteria, and existing integrations retain their access, isolation, and compatibility guarantees.

## Keeping the documents aligned

- Change behavior or acceptance criteria in the PRD.
- Change phase assignments, sequencing, and delivery status here, referencing PRD IDs rather than copying their detailed criteria.
- Update research when a dependency decision needs fresh evidence.
- Mark a milestone complete only after its deliverables and validation evidence exist. Draft documentation is not implemented functionality.
