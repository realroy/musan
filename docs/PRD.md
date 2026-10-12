# Musan — API access management

Status: requirements draft v0.2, for discussion; not an approved implementation plan.

Research date: 2026-10-12.

Working name: Musan, taken from the repository name.

Delivery phases, sequencing, milestones, and completion gates live in [ROADMAP.md](../ROADMAP.md). This PRD defines product behavior and acceptance criteria.

Repository workflow and agent instructions live in [AGENTS.md](../AGENTS.md).

## 1. Product intent

Musan helps application teams expose capabilities to external developers, services, and AI clients through APIs, webhooks, and MCP, with consistent access policies and operational visibility.

The product should integrate with applications across languages, frameworks, libraries, and tools. Application teams keep ownership of their business logic, identity system, and resource ownership checks. Musan supplies reusable access management capabilities and integration contracts.

### Confirmed by the project brief

- Start with a PRD and research existing software, including Unkey.
- Phase 1 includes API access management, webhooks, and MCP.
- Design for integration across languages and ecosystems.
- Use a monorepo with Turborepo.
- This stage produces documentation only; no implementation code or scaffolding.

### Working assumptions requiring a product decision

- Initial consumers are backend developers integrating Musan into existing applications, rather than infrastructure teams replacing a gateway.
- “Webhooks” initially means outgoing application events, including authorized subscriptions and reliable delivery. Receiving arbitrary third-party webhooks is a separate scope decision.
- “MCP” initially means exposing explicitly registered application tools with access enforcement. An MCP server for administering Musan is a separate scope decision.
- Simple tenant-scoped permissions are sufficient initially; complex relationship-based policies can use an integration later.
- Deployment mode, initial languages, distribution license, and hosted offering remain open. No database, queue, implementation language, or cloud vendor is selected.

## 2. Problem and product hypothesis

An application exposing an API needs credential lifecycle management, scoped permissions, tenant isolation, limits, and useful diagnostics. Adding webhook subscriptions and MCP tools introduces different credentials, delivery behavior, and protocol requirements. Implementing each interface independently risks inconsistent access and repeated integration work.

**Hypothesis:** teams will adopt a reusable integration layer if they can define application permissions once, apply them across these interfaces, and retain control over application data and business rules. This hypothesis needs validation with adopter interviews; competitor documentation does not establish customer demand.

Example: a customer integration may read orders through an API, subscribe to order updates, and use an MCP tool to inspect an order. All three must respect the same tenant and order-read grant. A read-only grant must never allow a write tool or another tenant's events.

## 3. Users and jobs

| User | Job to be done |
| --- | --- |
| Application developer | Add access management to an existing application without rewriting business logic or adopting a particular web framework. |
| Application administrator | Issue and restrict integration credentials, manage subscriptions, and revoke access. |
| Integration developer | Obtain a credential, understand permissions and limits, consume events, and diagnose failures. |
| Operator | Investigate denials and failed deliveries, rotate secrets, and recover from outages. |
| MCP client user | Grant an AI client limited access to application tools through an interoperable authorization flow. |

Musan administrators and external application consumers are separate trust boundaries. Existing host authentication may identify an administrator; it must not automatically grant that administrator access to every application's data.

## 4. Product scope

Priority **Must** identifies required product behavior. **Should** identifies proposed enhancements. These priorities are proposals, not evidence of a previously agreed backlog. Release assignments and sequencing are defined in [the roadmap](../ROADMAP.md#phase-1--first-usable-release).

| Capability | Product boundary |
| --- | --- |
| API access | API-key lifecycle, tenant/environment boundaries, named permissions, request admission limits, safe decision results. |
| Webhooks | Outgoing event subscriptions, per-endpoint signing, durable acceptance, retries, inspection, controlled replay. |
| MCP | Protect explicitly registered application tools over remote HTTP; integrate an existing OAuth authorization server. |
| Integration | Language-neutral contracts and conformance cases; a small, explicitly selected first set of language integrations. |
| Management | Management operations usable by a host application; a standalone dashboard is optional. |
| Operations | Access audit records, delivery diagnostics, documented failure and consistency behavior. |

### Outside the current product scope

- Replacing application login, SSO, or a complete identity provider.
- Implementing an OAuth authorization server, consent product, or token exchange system from scratch.
- A full gateway, API routing platform, WAF, service mesh, or developer portal.
- Billing, subscriptions, payment processing, and financial-grade usage metering.
- General-purpose workflow orchestration or inbound webhook routing.
- Automatically turning every application route into an MCP tool.
- MCP administration tools, agent orchestration, and additional MCP features beyond the chosen tool integration.
- Native integrations for every language or framework at launch.
- A globally distributed hosted platform, multi-region guarantees, or an enterprise policy engine.

These boundaries can be changed during review. Broad integration reach is a long-term requirement; supported integrations and delivery milestones are specified in the roadmap.

## 5. Shared domain and access rules

| Concept | Meaning |
| --- | --- |
| Application | A host application's integration boundary. |
| Tenant | A customer or organization boundary inside an application; distinct from a deployment or workspace. |
| Environment | An isolated context such as development, staging, or production. |
| Principal | An integration, service, or user whose permissions can be evaluated. |
| Credential | A way to authenticate a principal; API keys, OAuth access tokens, and webhook signing secrets have different purposes. |
| Permission | A named application action such as `orders:read`, mapped explicitly to routes, tools, or subscription eligibility. |
| Resource context | Trusted application-supplied ownership or resource attributes used for a decision. |
| Grant | A bounded assignment of permissions to a principal in a tenant and environment. |
| Limit | An admission rate or usage allowance with explicit scope and accounting rules. |
| Subscription | A permitted relationship between a tenant, event types, and an endpoint. |
| Event / delivery / attempt | The immutable application event, its delivery to an endpoint, and one HTTP transmission attempt. |

Proposed rules:

1. Access is denied unless an explicit grant and applicable boundary checks allow it.
2. Authentication success alone is insufficient. Tenant, environment, permission, and resource ownership must also pass.
3. The application supplies trusted resource context. A caller-provided tenant identifier cannot establish ownership by itself.
4. Permissions map to application capabilities rather than being inseparably tied to URL paths or tool names.
5. Credentials do not silently become interchangeable: an API key is not automatically a standards-compliant MCP OAuth token, and a webhook signature does not authorize API calls.
6. Policy evaluation returns a machine-readable decision and safe reason code. Sensitive policy details are available only to authorized operators.
7. Management operations enforce their own permissions and boundaries.

## 6. Functional requirements and acceptance criteria

### 6.1 Application integration and management

| ID | Priority | Requirement and observable acceptance |
| --- | --- | --- |
| INT-01 | Must | Provide framework-neutral contracts for management, credential verification, authorization, events, and error results. A host can integrate through documented contracts without importing a framework-specific adapter. The local-call versus remote-call form depends on the deployment decision. |
| INT-02 | Must | Integrate with an existing identity and administrator authentication system. A sample adopter can manage credentials without migrating its users to Musan. |
| INT-03 | Must | Keep framework bindings thin. Identical policy inputs produce equivalent decisions across supported integrations, including denial cases. |
| INT-04 | Must | Document supported languages, runtime versions, transports, and limitations. Supported integrations must pass shared contract conformance cases; the initial language selection and release coverage are defined in the roadmap. |
| INT-05 | Must | Separate trusted management authority from consumer credentials. A normal consumer key cannot mint keys, change grants, or inspect another principal's deliveries. |
| INT-06 | Should | Offer embeddable management UI components or a reference UI after management contracts stabilize. A full standalone dashboard is not a release dependency. |

### 6.2 API credentials and permissions

| ID | Priority | Requirement and observable acceptance |
| --- | --- | --- |
| KEY-01 | Must | Create, list metadata, expire, revoke, and rotate API keys bound to an application, tenant, environment, and principal. A revoked or expired key is rejected under the documented consistency guarantee. |
| KEY-02 | Must | Reveal a newly issued API-key secret only at issuance. Persist a nonrecoverable verifier; list, audit, and diagnostic operations never return the full secret. Store webhook signing secrets securely with encryption because signing requires usable key material. |
| KEY-03 | Must | Permit multiple credentials per principal with individually bounded permissions. Rotation supports a configurable, bounded overlap; revoking one credential does not unintentionally revoke another. |
| AUTH-01 | Must | Check explicit permissions and trusted resource context before executing protected work. Missing grants, tenant mismatch, and environment mismatch deny access. |
| AUTH-02 | Must | Define grant-update, credential-revocation, and principal-disable behavior across caches and replicas. Document the maximum delay, including requests already executing. OAuth token validity and application grant revocation are separate mechanisms. |
| AUTH-03 | Must | Support safe denial categories for invalid credential, insufficient permission, boundary mismatch, limit exceeded, and unavailable decision dependency. Public errors do not disclose resource existence across tenants. |
| AUTH-04 | Should | Support named permission bundles for common integration roles. Arbitrary policy expressions and relationship graphs are deferred. |

### 6.3 Limits and usage

| ID | Priority | Requirement and observable acceptance |
| --- | --- | --- |
| LIM-01 | Must | Support configurable request admission rates and fixed-period usage allowances, with explicit principal/tenant/environment scope. API requests and MCP tool invocations can share a configured budget. |
| LIM-02 | Must | Define counting units and whether denials or failed business operations consume allowance. Proposed initial rule: admitted operations consume one unit; denied operations do not. Retrying an operation does not imply a free allowance or business idempotency. |
| LIM-03 | Must | Define atomic admission and bounded overshoot, if any, for multiple application instances. Concurrent tests must demonstrate the published guarantee; an SDK's process-local counter alone cannot promise a global limit. |
| LIM-04 | Must | Return retry guidance where meaningful and expose remaining allowance safely. HTTP rate denial maps to 429; MCP errors follow the selected protocol and transport conventions. |
| LIM-05 | Should | Add weighted operation costs and richer usage analytics after basic admission accounting is stable. |

Webhook delivery attempts use separate operational limits so retries do not silently consume a consumer's API allowance.

### 6.4 Outgoing webhooks — provisional interpretation

| ID | Priority | Requirement and observable acceptance |
| --- | --- | --- |
| WH-01 | Must | Create, inspect, update, disable, and remove tenant-scoped endpoints and event subscriptions. A caller cannot subscribe to events beyond its grants. Applications explicitly register allowed event types and payload schemas. |
| WH-02 | Must | Give events stable identifiers and schema versions; preserve the event identity through retries and replay. Distinguish event, delivery, and attempt IDs in diagnostics. |
| WH-03 | Must | Promise delivery processing only after durable acceptance. Accepted pending work survives worker restart. Document the host's responsibility for atomically committing business changes and emitting events; do not imply automatic transactional capture. |
| WH-04 | Must | Provide at-least-once delivery within a documented retention/retry window. Duplicates are possible, ordering is not guaranteed initially, and consumers need idempotency. Exhausted deliveries enter an inspectable terminal state. |
| WH-05 | Must | Sign the exact outgoing body and authenticated event metadata using a documented Standard Webhooks-compatible profile. Provide receiver verification guidance, timestamp tolerance, per-endpoint secrets, and bounded secret-rotation overlap. |
| WH-06 | Must | Use bounded backoff with jitter, request timeouts, retry classification, and response-size limits. Define 2xx success, retryable timeouts/429/5xx, treatment of other statuses, and capped handling of Retry-After. |
| WH-07 | Must | Restrict destinations to authorized HTTPS endpoints. Reject private, loopback, link-local, and metadata-service destinations; revalidate resolved destinations at connection time; redirects must not bypass restrictions. Local development exceptions must be explicit. |
| WH-08 | Must | Allow authorized operators to inspect redacted delivery history and replay failed deliveries. Replay preserves event identity and creates a distinguishable new attempt history. |
| WH-09 | Must | Recheck subscription status and delivery eligibility before dispatch and replay. Revocation cancels pending unauthorized deliveries under the documented delay; already transmitted data cannot be recalled. Any queued payload requiring resource-level filtering uses a host-supplied eligibility check. |
| WH-10 | Should | Add test-event delivery and lifecycle notifications for repeatedly failing endpoints. A disabled endpoint must not resume delivery silently. |

The delivery engine can be supplied by a proven dependency or provider. The requirements describe observable behavior; they do not require building a queueing system. The signing reference is the [Standard Webhooks specification](https://github.com/standard-webhooks/standard-webhooks/blob/main/spec/standard-webhooks.md), which covers message metadata, exact-body signing, verification, and rotation. Retry, retention, and authorization behavior above are Musan product proposals.

### 6.5 MCP application access — provisional interpretation

The researched baseline is MCP **2026-07-28**, currently identified by the official versioning page. Recheck and pin a supported version/client matrix before implementation; older clients are supported only when explicitly selected. [MCP versioning](https://modelcontextprotocol.io/docs/2026-07-28/learn/versioning).

Protected remote HTTP MCP uses the protocol's OAuth resource-server model, discovery, and audience-bound access tokens. Local stdio retrieves credentials through its execution environment rather than the HTTP OAuth flow. These are protocol facts; the permission mappings below are Musan proposals. [MCP authorization](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization).

| ID | Priority | Requirement and observable acceptance |
| --- | --- | --- |
| MCP-01 | Must | Expose only explicitly registered application tools with input/output schemas and mapped permissions. Registering an API route alone never publishes an MCP tool. |
| MCP-02 | Must | Support protected remote HTTP with an existing compatible OAuth authorization server, resource metadata, and discovery/challenge behavior for the selected MCP version. An agreed pair of independent clients completes the authorization flow. |
| MCP-03 | Must | Validate access tokens, trusted issuer, intended audience, expiry, and scopes before protected processing. Map validated identity to a trusted application principal and tenant. A token intended for another service is rejected. Token verification may use JWT validation or introspection as appropriate to the provider. |
| MCP-04 | Must | Apply the shared permission, boundary, resource, and limit checks on every invocation before executing its handler. Tool visibility is filtered by the caller's grants, but hidden tools still reject direct unauthorized calls. Authorization cannot rely on connection state alone. |
| MCP-05 | Must | Return protocol-appropriate invalid-token and insufficient-scope responses. A narrowly scoped client cannot invoke a write tool; acquiring extra scopes still cannot bypass application grants or resource ownership. |
| MCP-06 | Must | Never forward the client's MCP bearer token to an unrelated upstream API. Upstream access requires a separately authorized credential. |
| MCP-07 | Must | Audit tool name, principal, tenant, policy result, and correlation ID while redacting arguments/results by default. Model-supplied arguments and tool annotations cannot establish permission or administrative authority. |
| MCP-08 | Should | Provide local stdio integration if chosen by adopters. Document environment/process trust and credential handling separately; do not claim it uses the remote OAuth flow. |

Authorization-dependent tool visibility is supported by the tools specification. Audience validation and the prohibition on upstream token passthrough follow the authorization security rules. [MCP tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools), [MCP authorization security considerations](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/security-considerations).

## 7. Nonfunctional requirements

| ID | Requirement | Proposed acceptance / decision needed |
| --- | --- | --- |
| NFR-01 | Tenant and environment isolation | All management, verification, tool, subscription, delivery, replay, and audit boundary cases pass shared negative conformance cases. |
| NFR-02 | Secret and payload protection | Secrets never appear in logs, URLs, or normal management responses. Define encryption, data retention, deletion, and diagnostic redaction responsibilities for the selected deployment. |
| NFR-03 | Explicit outage behavior | Protected new operations fail closed when no valid decision can be obtained. A documented bounded cache may be used only within its approved staleness policy. Outage results remain distinguishable from ordinary permission denial. |
| NFR-04 | Consistency | Proposed application grant/key revocation target: no new authorization after 5 seconds in a healthy deployment. Confirm feasibility per mode; do not promise OAuth token revocation without provider support. |
| NFR-05 | Performance | Define workload, hardware, region, cache state, concurrent load, and p95/p99 admission latency budgets before implementation. No latency or throughput claim is made yet. |
| NFR-06 | Durability | After webhook acceptance, process restart cannot silently lose pending deliveries. Retention, backup/restore, and dependency failure behavior must be specified for the selected mode. |
| NFR-07 | Observability | Correlate management changes, access decisions, and delivery attempts. Capture latency, denial reasons, backlog age, retry/exhaustion counts, and dependency errors without full business payloads by default. |
| NFR-08 | Compatibility | Version contracts and events; document supported runtimes and deprecation policy. Each shipped integration passes shared conformance cases. |
| NFR-09 | Portability | A consuming non-JavaScript application does not need Turborepo or Node.js solely to use its published Musan integration. Any runtime sidecar/service requirement is explicit. |
| NFR-10 | Supply and distribution | Select a license and support policy before publication. Review third-party deployment, license, and feature-edition constraints before selecting dependencies. |

The 5-second target is a proposed requirement to evaluate, not a benchmark or current capability.

## 8. Deployment and integration decision

This decision is intentionally unresolved. A Turborepo monorepo determines development organization; it does not determine how applications run Musan.

| Option | Benefit | Requirements impact |
| --- | --- | --- |
| Embedded library first | Fits the stated library intent; application owns runtime and data. | Define persistence, shared counters, background delivery workers, and cross-language packaging. Equivalent native implementations or bindings need a maintenance strategy. |
| Shared service with language SDKs | Centralizes credentials, policies, counters, and delivery behavior; HTTP contracts reach many languages. | Define service operation, network latency, availability, caching, and self-hosted/hosted distribution. This is more than an in-process library. |
| Both | Serves local and service-backed adopters. | Maintain two operational models and prove behavior parity. Do not require this in phase 1 without evidence of demand. |

Recommendation for review: keep contracts independent of frameworks and validate one runtime mode first. Do not choose both by default. Select the mode using adopter requirements for data ownership, network dependencies, global limits, webhook workers, and deployment simplicity.

## 9. Turborepo monorepo requirements

Turborepo is confirmed. Current official documentation supports mixed language workspaces, including experimental native Go, Python, and Rust support; languages without native support can use package-script wrappers. This is build orchestration, not automatic cross-language SDK generation or shared runtime semantics. [Turborepo multi-language guide](https://turborepo.dev/docs/guides/multi-language), [Turborepo 2.11 source documentation](https://github.com/vercel/turborepo/blob/main/apps/docs/content/blog/2-11.mdx).

Product acceptance requirements:

1. The monorepo supports development and validation of the selected language integrations under Turborepo orchestration.
2. Published integrations use native ecosystem distribution with explicit version compatibility; consumers do not clone the monorepo to use the product.
3. Framework-neutral integrations do not require a particular web framework, and supported integrations demonstrate equivalent contract behavior.

Operational rules for task orchestration, native toolchains, package boundaries, conformance validation, and caching are maintained in [AGENTS.md](../AGENTS.md#conventions-when-implementation-is-requested). Directory layout and technology selections remain design decisions.

## 10. Research implications and build/reuse choices

See [the software landscape research](research/api-access-landscape.md) for sourced comparisons of Unkey and adjacent products. Product feature claims are vendor documentation, not independently tested performance or evidence of universal suitability.

Decide capability by capability:

| Capability | Decision to evaluate |
| --- | --- |
| API-key lifecycle and limits | Reuse or integrate an existing access product such as Unkey versus owning an implementation. Compare runtime fit, deployment and edition constraints, latency, and revocation semantics. |
| Application permissions | Start with named grants and trusted host ownership checks; add a specialist authorization backend if policy complexity demands it. |
| Webhook delivery | Reuse established signing conventions and evaluate an existing delivery engine before building durable infrastructure. |
| MCP authentication | Integrate an OAuth provider and maintained protocol SDKs; keep application authorization enforcement under the shared Musan contract. |
| Gateway enforcement | Consider gateway adapters when adopters already use one; do not require a gateway for every integration. |

Potential differentiation is the integration experience and consistent application capability model across API calls, event subscriptions, and MCP tools. This is a positioning hypothesis, not a verified absence of competing functionality.

## 11. Product acceptance scenarios

All scenarios are future acceptance requirements; no implementation or runtime tests have been executed.

1. A developer integrates an existing application without changing its user identity system.
2. An administrator issues a tenant-scoped read credential. Allowed reads succeed; writes, another tenant's resources, and another environment's credentials fail.
3. Rotation supports a bounded overlap. Revocation stops new authorized requests within the agreed delay, including cached decisions.
4. Concurrent requests from multiple instances honor the published rate/allowance guarantee; API and MCP share a budget when configured.
5. A permitted webhook subscription receives signed events. Restart, timeout, rate denial, exhausted retries, and replay behave as documented without cross-tenant disclosure.
6. Disabling a principal or subscription prevents pending unauthorized deliveries and replay. No claim is made about recalling events already sent.
7. A selected independent MCP client authenticates through a compatible OAuth provider. An invalid-audience token fails; a read-only user cannot directly invoke a hidden write tool.
8. The same access conformance scenarios pass through two selected language integrations, with safe equivalent reason codes.
9. Operators can explain a denial or failed delivery from redacted diagnostics without obtaining the consumer's secret.
10. A clean consuming application uses a published integration with only the documented runtime dependencies.

## 12. Success measures and validation

Proposed product validation before a broad release:

- Interview at least three candidate adopter teams about current access management, language stack, and webhook/MCP needs.
- Demonstrate integration in two real or representative existing applications across the selected languages.
- Measure time to first protected API call, first verified webhook, and first authorized MCP tool invocation. Establish targets after the initial adopter baseline.
- Track integration friction: framework-specific workarounds, required infrastructure, unresolved policy ambiguity, and SDK behavior differences.
- Treat isolation, unauthorized handler execution, and secret leakage as release-blocking failures in the agreed acceptance cases.
- Do not use synthetic throughput alone as evidence that the product solves adopter needs.

## 13. Decisions needed before implementation

| Decision | Current state | Why it matters |
| --- | --- | --- |
| Runtime mode | Open: embedded, shared service, or both | Sets persistence, worker ownership, availability, and SDK architecture. |
| Webhook meaning | Assumed outgoing application events | Receiving providers or lifecycle-only notifications changes the product boundary. |
| MCP meaning | Assumed protected application tools | Administrative tooling is a different user journey and authority model. |
| First languages/frameworks | Open; proposed two-language validation | Determines deliverables, maintainers, native packaging, and compatibility work. |
| Initial adopter and workload | Open | Needed for latency, throughput, durability, and adoption targets. |
| Tenant/principal model | Proposed application + tenant + environment | Validate against real multi-application and customer requirements. |
| Permissions complexity | Proposed named grants plus host ownership checks | Determines whether a specialist policy engine belongs in phase 1. |
| OAuth provider/client matrix | Open | Needed to prove actual MCP interoperability and token lifecycle behavior. |
| Build versus dependency | Open per capability | Changes operating costs, licensing, deployment burden, and ownership. |
| Delivery/revocation targets | Proposals, numeric retry/retention budgets open | Needed to make operational guarantees testable. |
| Distribution and license | Open | Library, self-hosted service, and managed offering carry different commitments. |

These decisions feed the requirements and design phase in [the roadmap](../ROADMAP.md#phase-0--requirements-and-design). No architecture proposal in this draft authorizes implementation.
