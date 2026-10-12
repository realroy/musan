# Instructions for agents working on Musan

## Current scope

Musan is in requirements and research. Work on documentation only until the user requests implementation. Do not scaffold packages, install project dependencies, generate application code, or deploy a service as part of documentation work.

Confirmed direction: application API access management, webhooks, and MCP in phase 1; integration across languages and frameworks; a Turborepo monorepo. Runtime mode, initial languages, providers, persistence, package manager, and distribution license remain undecided. Preserve that distinction when editing documents.

## Document responsibilities

| Document | Owns |
| --- | --- |
| [PRD](docs/PRD.md) | Product purpose, users, scope, behavior, requirement IDs, and acceptance criteria. |
| [Roadmap](ROADMAP.md) | Phase scope, sequencing, deliverables, dependencies, completion gates, and delivery status. |
| [Research](docs/research/api-access-landscape.md) | Dated primary-source evidence, comparisons, and build-versus-reuse implications. |
| [AGENTS.md](AGENTS.md) | Instructions for agents working in this repository. |
| [README](README.md) | Concise project introduction and navigation. |

- Put detailed phase plans in the roadmap and refer to PRD requirement IDs. Keep product acceptance criteria in the PRD.
- Mark proposals, assumptions, and unresolved decisions explicitly. Do not convert roadmap candidates into commitments.
- Preserve requirement IDs when reorganizing content; update links and references when headings change.
- Separate documented product claims from Musan proposals and our interpretation. Cite primary sources and date research; do not claim that vendor documentation proves runtime behavior or customer demand.
- Update all affected documents when changing scope. Mark work complete only when its deliverables and validation evidence exist.

## Repository workflow

- Inspect the current files and Git status before editing. Preserve unrelated changes and ongoing work.
- Keep changes scoped to the user's request. Do not choose an undecided technology or runtime mode as a side effect of editing documentation.
- Use relative Markdown links within the repository. Check affected links, heading anchors, requirement references, and whitespace after changes.
- Run `git diff --check` for tracked changes and inspect new files as well. There is no application build or test suite yet; do not report runtime verification for documentation-only work.
- Report what changed, what was checked, and whether changes were committed or pushed.

## Conventions when implementation is requested

These rules describe future repository work; they do not authorize implementation or define a directory scaffold.

- Use Turborepo to orchestrate applicable build and validation tasks while respecting package dependencies.
- Preserve language-native toolchains, lockfiles, packaging, and registry distribution. Evaluate experimental workspace discovery before requiring it in the build.
- Keep framework-specific dependencies out of framework-neutral consumers. Keep framework and MCP adapters thin and verify relevant package boundaries with suitable tooling.
- Contract changes must trigger conformance validation for affected language integrations and examples.
- Include contracts, relevant lockfiles, configuration, and toolchain versions in cache inputs. Exclude secrets and private runtime data from cached outputs.
- Run checks appropriate to the affected behavior and report actual results. Shared access behavior needs equivalent allow and deny cases across supported integrations.

The PRD's proposed logical areas are contracts, runtime, language integrations, framework/MCP adapters, examples, documentation, and an optional management application. Directory names and package boundaries must follow the eventual design decision, rather than being treated as already selected.

<!-- context7 -->
## Current library and tool documentation

Use the `ctx7` CLI to fetch current documentation whenever the user asks about a library, framework, SDK, API, CLI tool, or cloud service -- even well-known ones like React, Next.js, Prisma, Express, Tailwind, Django, or Spring Boot. This includes API syntax, configuration, version migration, library-specific debugging, setup instructions, and CLI tool usage. Use even when you think you know the answer -- your training data may not reflect recent changes. Prefer this over web search for library docs.

Do not use for: refactoring, writing scripts from scratch, debugging business logic, code review, or general programming concepts.

### Steps

1. Resolve library: `npx ctx7@latest library <name> "<user's question>"` — use the official library name with proper punctuation (e.g., "Next.js" not "nextjs", "Customer.io" not "customerio", "Three.js" not "threejs")
2. Pick the best match (ID format: `/org/project`) by: exact name match, description relevance, code snippet count, source reputation (High/Medium preferred), and benchmark score (higher is better). If results don't look right, try alternate names or queries (e.g., "next.js" not "nextjs", or rephrase the question)
3. Fetch docs: `npx ctx7@latest docs <libraryId> "<user's question>"`
4. Answer using the fetched documentation

You MUST call `library` first to get a valid ID unless the user provides one directly in `/org/project` format. Use the user's full question as the query -- specific and detailed queries return better results than vague single words. Do not run more than 3 commands per question. Do not include sensitive information (API keys, passwords, credentials) in queries.

For version-specific docs, use `/org/project/version` from the `library` output (e.g., `/vercel/next.js/v14.3.0`).

If a command fails with a quota error, inform the user and suggest `npx ctx7@latest login` or setting `CONTEXT7_API_KEY` env var for higher limits. Do not silently fall back to training data.
Run Context7 CLI requests outside Codex's default sandbox. If a Context7 CLI command fails with DNS or network errors such as ENOTFOUND, host resolution failures, or fetch failed, rerun it outside the sandbox instead of retrying inside the sandbox.
<!-- context7 -->
