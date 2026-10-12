# API access management landscape

Research date: **2026-10-12**. Purpose: inform Musan's first PRD, without selecting an implementation or commercial dependency.

## Evidence and scope

This is a focused comparison of five representative products, not an exhaustive market survey. **Documented** means a capability described by its vendor or project; it does not mean independently benchmarked or production-tested here. **Implication** is our interpretation for Musan. No services were deployed, SDKs exercised, prices compared, or compliance claims validated.

Current documentation was retrieved with Context7, resolving each library before fetching its documentation: Unkey, Kong Gateway, OpenFGA, Svix, and Auth0. First-party documentation and repositories were then consulted for specific claims and license boundaries. Some Auth0 quickstart URLs were unavailable through the browser; its accessible MCP documentation and documentation-source repository provide the evidence below.

The working interpretation is that phase 1 covers application API access, outbound application webhooks, and access to application MCP tools. These webhook and MCP interpretations remain product decisions. An MCP server for administering an access-management vendor is different from protecting the integrating application's MCP server.

## Comparison

| Product | Documented focus and integration | Implication for Musan |
| --- | --- | --- |
| Unkey | API keys, permissions/RBAC, rate limiting, analytics, and a broader deployment/gateway platform; HTTP API and SDK integration. [Project overview](https://github.com/unkeyed/unkey) | Closest reference for credentials and API consumption controls. Define why Musan should exist alongside it. |
| Kong Gateway | Authentication, access controls and rate limiting at a gateway; separately licensed MCP plugins. [Key Auth](https://developer.konghq.com/plugins/key-auth/), [ACL](https://developer.konghq.com/plugins/acl/), [MCP OAuth2](https://developer.konghq.com/plugins/ai-mcp-oauth2/) | A gateway is a viable alternative when centralized network enforcement is acceptable. Musan should justify any application-level integration requirement. |
| OpenFGA | Fine-grained authorization through HTTP/gRPC, language SDKs and relationship models. [Project overview](https://github.com/openfga/openfga) | Potential authorization provider or design reference, rather than a credential-and-delivery subsystem by itself. |
| Svix | Webhook delivery service and self-hostable server, with API clients and signature verification libraries. [Project overview](https://github.com/svix/svix-webhooks) | Strong reference or reusable provider for webhook operations; reliability requires more than an HTTP request helper. |
| Auth0 Auth for MCP | OAuth identity, delegated permissions and MCP integration. [Why Auth for MCP](https://auth0.com/ai/docs/mcp/why-auth-for-mcp) | Prefer compatibility with established authorization servers over inventing an identity system. |

## Unkey: the closest baseline

**Documented.** Unkey currently describes issuing, verifying and revoking API keys; per-key permissions and roles; rate limiting; analytics and audit logs. Its current positioning also includes API deployment and gateways, so describing it solely as an API-key library understates the product. The README permits self-hosting under its license, but this research does not establish operational simplicity or hosted/self-hosted feature parity. [Unkey overview](https://github.com/unkeyed/unkey)

Its Python quickstart documents key expiration, credits with refill, multiple rate limits, metadata and association with an external user identifier. Full key material is returned at creation. Its v2 migration documentation moves API verification authority into root-key permissions and documents role operations. The SDK repository lists TypeScript, Go and Python API SDKs. [Python quickstart](https://www.unkey.com/docs/quickstart/apis/python), [v2 key migration](https://www.unkey.com/docs/api-reference/v1/migration/keys), [SDK repository](https://github.com/unkeyed/sdks)

Context7 also retrieved an official example connecting an MCP client to Unkey's rate-limiting MCP endpoint with a root key. This is evidence of an administration/integration interface, not sufficient evidence that every integrating application's MCP OAuth boundary is supplied automatically. [Unkey Windsurf documentation source](https://github.com/unkeyed/unkey/blob/main/docs/product/ai-code-gen/windsurf.mdx)

**Implication.** API key CRUD and basic rate limiting are established capabilities. Musan's proposed differentiator should be evaluated as a consistent application integration contract across API operations, webhook subscriptions and MCP permissions. Cross-language support alone is insufficient differentiation: Unkey already documents several SDKs. Explicitly evaluate wrapping Unkey against operating Musan-owned credential storage; no dependency is selected here.

## Kong: enforce access at the network boundary

**Documented.** Key Auth associates credentials with authenticating entities; ACL policies restrict access; rate limiting can apply at routes or services and identify authenticated Consumers. Rate counters can use local, cluster or Redis strategies, with documented accuracy and topology tradeoffs. [Key Auth](https://developer.konghq.com/plugins/key-auth/), [ACL](https://developer.konghq.com/plugins/acl/), [Rate Limiting](https://developer.konghq.com/plugins/rate-limiting/)

Kong distinguishes OSS traditional/database-backed and DB-less deployment from Enterprise's additional hybrid mode. Its AI MCP OAuth2 plugin validates tokens for the target resource and supports external authorization servers. The plugin page explicitly labels it **Tech Preview**, **AI License Required**, and part of AI Gateway Enterprise; these capabilities must not be attributed to the Apache-licensed OSS gateway indiscriminately. [OSS/Enterprise deployment distinction](https://developer.konghq.com/gateway/upgrade/migrate-oss-to-ee/), [MCP OAuth2 plugin](https://developer.konghq.com/plugins/ai-mcp-oauth2/)

**Implication.** Gateway integration reduces language coupling because enforcement occurs before the application. Application adapters still matter when permission checks need domain resources or when upstream services can be reached directly. The PRD should define the enforcement boundary and bypass assumptions. Evaluate gateway coexistence; outbound durable webhook delivery remains a separately assessed responsibility, not an inferred gateway guarantee.

## OpenFGA: permission decisions independently of credentials

**Documented.** OpenFGA exposes HTTP/gRPC APIs, supports several storage backends, and lists SDKs for Java, Node.js, Go, Python and .NET. It can also be embedded as a Go library. Its quickstart warns that default in-memory storage is ephemeral. [Project overview and deployment guidance](https://github.com/openfga/openfga)

Its MCP documentation models tool access, groups, roles, temporal grants and resource-level permissions. The integration flow starts with separate identity-provider authentication, filters discoverable tools, and checks authorization again during execution. [MCP authorization](https://openfga.dev/docs/modeling/agents/mcp-authorization)

**Implication.** Credential validity and permission to perform an operation are separate decisions. Musan should map an authenticated principal to a stable tenant/action/resource decision contract. OpenFGA is a possible provider for richer relationships; phase 1 need not implement a new policy language. Its documented authorization role does not establish API key lifecycle, consumption counters or durable webhook delivery as part of the same service.

## Svix: webhook reliability and verification

**Documented.** Svix offers a webhook sending service and a self-hostable server. Its repository lists SDKs covering API access and webhook verification across Go, Python, TypeScript/JavaScript, Java, Kotlin, Ruby, C#, Rust and PHP. The self-hosted README distinguishes server operation from the hosted service and documents operational dependencies. [Repository and SDK matrix](https://github.com/svix/svix-webhooks)

Delivery retries use exponential backoff. Successful delivery requires a timely 2xx response; failures exhaust into a failed state with operational notifications, and customers can retry or recover messages. Its idempotency documentation explicitly describes **at-least-once** delivery, reused message identifiers across attempts, and receiver-side deduplication. Idempotent producer API requests and deduplicated receiver processing are distinct. [Retries and recovery](https://docs.svix.com/retries), [Idempotency](https://docs.svix.com/idempotency)

Manual verification signs the message identifier, timestamp and exact raw body using HMAC-SHA256; documentation recommends constant-time comparison and timestamp tolerance checks. [Signature verification](https://docs.svix.com/receiving/verifying-payloads/how-manual)

**Implication.** Musan's webhook requirement should cover durable acceptance, retry policy, recovery, delivery visibility, signing-secret rotation and receiver guidance. Reusing delivery infrastructure is a credible option. Musan would still need to define application events, tenant ownership, subscription permissions and whether disabling an integration cancels queued deliveries. Do not promise exactly-once effects across arbitrary HTTP receivers.

## Auth0: standards-based MCP access

**Documented.** Auth for MCP describes OAuth access-token validation, identity-bound scopes, consent and metadata discovery. Its documentation-source quickstart describes securing server requests with access tokens and enforcing role-based tool permissions. Its Inspector guide demonstrates Streamable HTTP and distinguishes static client registration from optional dynamic registration. [Why Auth for MCP](https://auth0.com/ai/docs/mcp/why-auth-for-mcp), [Server authorization source](https://github.com/auth0/docs-v2/blob/main/main/snippets/ai/mcp/get-started/server-authorization.mdx), [MCP Inspector guide](https://auth0.com/ai/docs/mcp/guides/test-your-mcp-server-with-mcp-inspector)

**Implication.** Musan should support an external authorization server and enforce operation/resource permissions after authentication. OAuth support should have explicit issuer, audience, token-lifetime and discovery requirements. Exact conformance belongs against a pinned MCP specification, separately from a vendor tutorial. This comparison does not establish Auth0 as a self-hostable library or as a durable webhook delivery provider; neither capability is a selection assumption.

## Verified license boundaries

| Artifact | Verified license/scope |
| --- | --- |
| Unkey main repository | AGPLv3 default, with package-specific and third-party exceptions stated in the root license. [License](https://github.com/unkeyed/unkey/blob/main/LICENSE) |
| Unkey SDK repository | MIT; this does not relicense the server. [SDK license](https://github.com/unkeyed/sdks/blob/main/LICENSE) |
| Kong OSS repository | Apache-2.0; separately licensed enterprise/AI features are outside this conclusion. [OSS license](https://github.com/Kong/kong/blob/master/LICENSE) |
| OpenFGA server repository | Apache-2.0. [License](https://github.com/openfga/openfga/blob/main/LICENSE) |
| Svix webhook repository | MIT as declared by the official README; hosted service terms are separate. [License declaration](https://github.com/svix/svix-webhooks#license) |
| Auth0 hosted platform | No open-source platform license conclusion made. Inspect individual SDK licenses if selected. |

These are observed artifact licenses, not legal conclusions about a proposed combination. Recheck the exact revision and component before reuse.

## Requirements to carry into the PRD

The following are **proposals**, not evidence that competitors lack them:

1. A shared principal, tenant, action and resource vocabulary across API checks, webhook subscriptions and MCP execution.
2. Explicit credential issuance, expiration, revocation and rotation semantics, with administrator credentials separated from application integration credentials.
3. Language-neutral contracts and conformance fixtures before broad SDK expansion; distinguish protocol compatibility from officially maintained adapters.
4. Defined outage behavior, revocation freshness, permission-cache lifetime and distributed rate-limit consistency.
5. Durable webhook acceptance, finite retries, recovery, signature verification and deduplication contracts, including authorization changes while events are queued.
6. MCP identity-provider interoperability, separate tool discovery and execution checks, resource permissions and an auditable user/client context.
7. A build-versus-reuse comparison using the same acceptance cases, integration effort, deployment burden and license constraints.

A plausible Musan opportunity is coordinated integration behavior across these surfaces. This remains a hypothesis requiring feedback from target application developers, rather than a verified market-wide gap.
