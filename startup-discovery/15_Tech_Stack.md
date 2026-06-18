# 15_Tech_Stack.md

# Technology Stack Reference Document

## Product

PreFlight — MCP & Agent Permission Attack-Surface Scanner

## Document Status

Derived from `11_Final_Startup_Selection.md` (final decision), `12_Product_Requirements_Document.md` (functional/non-functional requirements), `13_System_Architecture.md` (system architecture, ADR-001–ADR-009), and `14_AI_Architecture.md` (AI/ML architecture, AI-ADR-001–AI-ADR-008). This document is the **official, consolidated technology reference** for the entire 15-month build. Where 13/14 already made a binding decision, it is **restated here for completeness** with the full Selected/Purpose/Why/Alternatives/Advantages/Disadvantages/Cost/Learning-Curve/Scalability breakdown the user requested — these restatements do not change the prior decision. Where this document makes a **new** decision (dependency management, testing stack, JWT/password-hashing libraries, error tracking, IaC tool, etc.), it is flagged explicitly and numbered as a **TECH-ADR** (Section 14).

Two corrections to prior documents are made explicit up front, in the same spirit as `14_AI_Architecture.md`'s VRAM correction:

1. **`13_System_Architecture.md` §6 proposed `python-jose` for JWT handling and `passlib` for password hashing.** Both are corrected in Section 6.1 below — `python-jose` has a history of algorithm-confusion CVEs and reduced maintenance velocity; `passlib` has been unmaintained since 2020 and has known breakage against modern `bcrypt` releases. For a security product, shipping either in the auth path is a credibility risk if surfaced by a security researcher reviewing PreFlight's own stack — a highly plausible scenario given PreFlight's positioning.
2. **Several supporting categories the user explicitly requested (Dependency Management, full Testing Stack, parts of Monitoring, IaC tooling, Vulnerability Scanning) were not fully specified in `13_System_Architecture.md` or `14_AI_Architecture.md`.** These are designed fresh in this document, following the same "boring tech, minimize operational surface area for a 3-person team" philosophy established in ADR-001 through ADR-009 and AI-ADR-001 through AI-ADR-008.

## Audience

All three engineers, for environment setup, dependency decisions, and as the canonical answer to "what do we use for X and why" for the remainder of the 15-month build (and for onboarding any future hire/co-founder/design partner who asks).

## Prepared By (Panel Roles)

Principal Software Architect · Principal AI Engineer · DevOps Architect · Cybersecurity Architect · Startup CTO · Cloud Solutions Architect · VC Technical Due Diligence Reviewer

---

# 0. Stack Philosophy (Carried Forward, Restated)

Every decision below is evaluated against the same three constraints that govern `13_System_Architecture.md` and `14_AI_Architecture.md`, in priority order:

1. **3-person team, 15-month timeline** — every additional technology, framework, or managed service is a recurring tax on a team that cannot afford specialists. "Boring," well-documented, widely-adopted tools beat novel ones unless the novel tool's advantage is large and directly serves a PRD goal (G1–G5, T1–T5).
2. **Consumer hardware for development** (RTX 2050 / 4GB VRAM, 24GB system RAM, i5 H-series — per AI-ADR-002's correction) — nothing in the local dev loop should assume a GPU workstation or an always-on cloud dev environment.
3. **"Fewer systems" as a recurring tie-breaker** — this document inherits and extends the pattern from ADR-001 (no Neo4j), ADR-005 (no second database), AI-ADR-005 (no vector database) and applies it to *every* new category introduced here: no managed auth platform unless justified, no dedicated eval framework unless justified, no second IaC tool, no second CI platform.

A secondary, VC/recruiter-facing principle also applies: **every technology choice should be defensible in a technical due-diligence conversation or a job interview** — i.e., mainstream enough that an acquirer's engineering team or a security-company interviewer recognizes it immediately, but applied with enough judgment (the corrections in this document) to demonstrate the team did not cargo-cult its stack.

---

# 1. Frontend Stack

## 1.1 Framework — React 18 + TypeScript + Vite

| Attribute | Detail |
|---|---|
| **Selected Technology** | React 18 + TypeScript, bundled/served via Vite |
| **Purpose** | SPA framework for `preflight-web` — the dashboard serving UC-3, UC-4, UC-5, UC-7 (`13_System_Architecture.md` §3) |
| **Why Selected** | Restated from `13_System_Architecture.md` §3. Team already has React experience (`01_Project_Framework.md` skills inventory); Vite's dev server/HMR is materially faster than the now-deprecated Create React App or a hand-rolled Webpack config; TypeScript gives compile-time safety across the graph/finding data structures that flow from Pydantic schemas (Section 2.4) through to Cytoscape rendering. |
| **Alternatives Considered** | **Next.js** — rejected: SSR/file-based routing/edge-rendering features are unnecessary for an authenticated SPA behind a REST API, and would add a Node server deployment target where a static S3+CloudFront bundle (`13_System_Architecture.md` §17.1) currently costs near-zero. **Vue 3** — rejected on skill-fit: zero prior team experience, and shadcn/ui-equivalent component ecosystems are React-first. **SvelteKit** — rejected: smaller component-library ecosystem (no Cytoscape/shadcn-equivalent maturity), and again a skill mismatch. |
| **Advantages** | Massive ecosystem and hiring pool (recruiter value, A1); Vite's speed materially improves the 3-person team's daily iteration loop; TS catches integration bugs between frontend types and the FastAPI OpenAPI schema before runtime; static export keeps hosting cost near-zero. |
| **Disadvantages** | React ecosystem churn requires periodic dependency-bump discipline (mitigated by `uv`/`npm` lockfiles, Section 2.5); client-side-only rendering means no SEO benefit (irrelevant for an authenticated dashboard, but relevant if a marketing site is later added — see Future Scalability). |
| **Cost Impact** | $0 direct licensing cost; hosting folded into S3 + CloudFront (~$1–2/month, Section 12). |
| **Learning Curve** | Low — team already knows React; Vite's migration from CRA-style tooling is near-zero friction. |
| **Future Scalability** | Scales as an SPA without architectural rewrite. If a separate marketing/landing site is needed (open-core adoption flywheel, `11_Final_Startup_Selection.md` §11), a small standalone Next.js or static-site-generator project is cleaner than retrofitting SSR into the dashboard. |

## 1.2 UI Library — Tailwind CSS + shadcn/ui

| Attribute | Detail |
|---|---|
| **Selected Technology** | Tailwind CSS (utility-first styling) + shadcn/ui (Radix UI primitives, distributed as owned source, not an npm dependency) |
| **Purpose** | Styling system and accessible component primitives for the routes/components specified in `13_System_Architecture.md` §3.1–3.2 (tables, dialogs, forms, `FindingCard`, `SimulationTranscriptViewer`, `RemediationDiffViewer`). |
| **Why Selected** | shadcn/ui ships components as source the team copies into `/components/ui` and owns outright — important for a security dashboard with bespoke, domain-specific components that need deep customization (color-coded risk scores, transcript diff rendering) without fighting a component library's theming API. Radix UI underpins shadcn/ui and provides WAI-ARIA-compliant accessibility "for free." |
| **Alternatives Considered** | **MUI (Material UI)** — rejected: heavier bundle, Material Design aesthetic doesn't fit a developer/security-tool product, and theming to match Cytoscape's risk-score palette is more friction than Tailwind's design-token approach. **Chakra UI** — good DX but less Tailwind-native; would mean maintaining two styling paradigms if Tailwind is used elsewhere (it is, per Cytoscape stylesheets and the global design system). **Ant Design** — enterprise-CRM aesthetic, heavy bundle, harder to restyle to a developer-tool look. |
| **Advantages** | Full ownership of component code — no surprise breaking changes from an upstream component-library major version; Radix accessibility baked in; Tailwind's utility classes integrate cleanly with any CSS-variable-driven theming used elsewhere, so shared design tokens transfer easily. |
| **Disadvantages** | "Dependency" is actually owned code — shadcn/ui component updates must be manually re-pulled via its CLI (no `npm update` semantics); Tailwind's utility-class verbosity in JSX can reduce readability for contributors unfamiliar with the convention (mitigated by Prettier's Tailwind class-sorting plugin). |
| **Cost Impact** | $0. |
| **Learning Curve** | Low–Medium — Tailwind's utility-first mental model takes roughly 1–2 weeks to become fluent; Radix primitives are mostly invisible to day-to-day component authoring. |
| **Future Scalability** | Scales well as the design system grows — a shared `tailwind.config.ts` plus a conventionally-organized `/components/ui` directory keeps things maintainable even as Phase 2 features (drift diff views, MSSP multi-project views, UC-8/UC-9) add new screens. |

## 1.3 State Management — TanStack Query + Zustand

| Attribute | Detail |
|---|---|
| **Selected Technology** | TanStack Query (server state) + Zustand (client/UI state) |
| **Purpose** | TanStack Query: fetching/caching/polling for projects, scans, findings, graph snapshots (`13_System_Architecture.md` §3.2's `ScanProgressTracker` polling pattern). Zustand: ephemeral UI state — selected graph node, sidebar/panel open state, active theme. |
| **Why Selected** | Clean separation of concerns: almost all of PreFlight's frontend data *is* server state (scans, findings, transcripts) with well-defined cache-invalidation triggers (a new scan invalidates the findings list; a sink-rule edit invalidates future scans' scoring) — exactly TanStack Query's design target, including built-in polling for the async scan-status flow. The remaining state (what's selected/open in the UI right now) is small and purely local — Zustand handles it with near-zero boilerplate. |
| **Alternatives Considered** | **Redux Toolkit + RTK Query** — rejected: more boilerplate (slices, reducers, selectors) for a 3-person team with no prior Redux experience, and RTK Query's feature set substantially overlaps TanStack Query's without a clear edge for this app's shape. **SWR** — viable alternative to TanStack Query, but has a narrower mutation/optimistic-update API surface, which matters for UC-6/UC-7's edit-and-immediately-reflect flows. **Plain React Context** — would require hand-rolling polling, retry, and cache-invalidation logic that TanStack Query provides — reinventing the wheel. |
| **Advantages** | TanStack Query's polling + `staleTime`/`refetchInterval` config is a direct fit for `ScanProgressTracker`; Zustand's ~1KB footprint and hook-based API impose almost no learning tax; both are fully TypeScript-typed. |
| **Disadvantages** | Two libraries = two mental models for new contributors (mitigated by a one-line team convention: "server data → TanStack Query; everything else → Zustand"); cache-invalidation discipline after mutations (scan triggers, sink-rule edits, simulation re-runs) requires deliberate query-key design — getting this wrong produces stale-UI bugs that are easy to miss in manual testing. |
| **Cost Impact** | $0. |
| **Learning Curve** | Low for Zustand (minutes). Medium for TanStack Query — cache-key namespacing and invalidation strategy is the one genuinely non-trivial concept; budget roughly a week of deliberate practice early in Month 1. |
| **Future Scalability** | Both scale to large apps without rewrites. TanStack Query's hierarchical query keys (`['projects', id, 'scans', scanId, 'findings']`) extend cleanly to Phase 2 resource types (drift diffs, MSSP multi-project dashboards, UC-8/UC-9) by adding new key namespaces, not new infrastructure. |

## 1.4 Graph Visualization — Cytoscape.js + cytoscape-dagre

| Attribute | Detail |
|---|---|
| **Selected Technology** | Cytoscape.js + `cytoscape-dagre` (layout) + `cytoscape-popper` (tooltips/popovers) |
| **Purpose** | `PermissionGraphView` — the interactive permission-graph canvas (UC-3), with risk-score color coding and attack-path highlighting (`13_System_Architecture.md` §3.2). |
| **Why Selected** | Restated from `13_System_Architecture.md` §3 — chosen specifically "for better performance on graphs up to ~1,000 nodes per NFR-1 and built-in graph layout algorithms." `cytoscape-dagre` provides a hierarchical layout that maps naturally to the Agent → Tool → DataResource → ExternalSink structure of the permission graph (`12_Product_Requirements_Document.md` §6.1). |
| **Alternatives Considered** | **react-flow** — explicitly rejected per `13_System_Architecture.md` §3: weaker rendering performance at the ~1,000-node/~5,000-edge ceiling (NFR-1), though it offers superior "React-native" component DX for smaller flow-builder-style UIs (not PreFlight's use case). **D3.js (raw)** — maximum flexibility, but the team would be hand-building a layout engine, interaction model, and pan/zoom from scratch — not justified for a 3-person team's timeline when Cytoscape provides all three out of the box. **vis-network** — older project, slower release cadence, fewer maintained layout plugins than Cytoscape's ecosystem. |
| **Advantages** | Proven at the relevant scale (used in network-security and bioinformatics tooling with similarly-sized graphs); declarative stylesheet-based styling maps directly onto risk-score color scales; Cytoscape's headless mode is usable server-side too — a future option for generating static graph images embedded in PDF reports (`13_System_Architecture.md` §13, ADR-007). |
| **Disadvantages** | Imperative API — bridging Cytoscape's instance lifecycle with React's render cycle requires a small wrapper hook/component (or `react-cytoscapejs`), an extra integration layer beyond "drop in a component"; at the upper end of NFR-1's 1,000-node bound, canvas rendering benefits from progressive-disclosure UX (collapse low-risk subgraphs by default) — a UX design task, not just a library decision, that should be planned for rather than discovered late. |
| **Cost Impact** | $0. |
| **Learning Curve** | Medium — Cytoscape's element/stylesheet/layout/event model is a distinct mental model from typical React component libraries; budget 1–2 weeks for the first integration (graph rendering + click-to-highlight attack paths). |
| **Future Scalability** | At graph sizes meaningfully beyond 1,000 nodes (large enterprise fleets / MSSP multi-client views, Phase 2–3, UC-8), the documented mitigation is server-side subgraph filtering/pagination, not a renderer swap. A WebGL-based renderer (e.g., Sigma.js) is the explicit Phase 3 fallback only if a genuine 10,000+-node requirement materializes — out of scope for the 15-month MVP. |

## 1.5 Authentication (Frontend) — In-Memory JWT + httpOnly Refresh Cookie + GitHub OAuth

| Attribute | Detail |
|---|---|
| **Selected Technology** | Access token held in React state/memory (never `localStorage`); refresh token in an `httpOnly` + `Secure` + `SameSite=Strict` cookie; GitHub OAuth via Authlib on the backend (Section 6.1) |
| **Purpose** | Authenticate web users for email/password and GitHub OAuth login flows (`13_System_Architecture.md` §3 security considerations, §6.2). |
| **Why Selected** | In-memory access tokens mitigate XSS-based token theft (a compromised dependency cannot read `localStorage`/`sessionStorage` for the token — it can still act within the page, but cannot exfiltrate a long-lived credential); the `httpOnly` refresh cookie survives page reloads without being readable by JS, mitigating the same class of attack while avoiding CSRF via `SameSite=Strict`. GitHub OAuth specifically reduces signup friction for the primary persona (AI Platform Engineer, `12_Product_Requirements_Document.md` §4.1) and pairs naturally with the GitHub Action CI integration (Module E). |
| **Alternatives Considered** | **Auth0 / Clerk / Supabase Auth (managed auth-as-a-service)** — would remove meaningful auth-implementation work, but (a) most free tiers cap at a few thousand MAU and become a recurring cost as the user base grows, (b) couples a security product to a third-party identity provider for *access to security findings* — a sensitivity profile most managed-auth vendors aren't specifically positioned for, and (c) the custom `organizations`/`memberships`/role schema (`13_System_Architecture.md` §6.1, ADR-006) doesn't map cleanly onto most managed-auth user models without workaround tables. **Pure `localStorage` JWT** — rejected outright on XSS-exposure grounds; this is a security product and should not ship the textbook anti-pattern token-storage approach. |
| **Advantages** | Full control of the session model (15-minute access tokens, rotating/revocable refresh tokens per `13_System_Architecture.md` §6); GitHub OAuth via Authlib (Section 6.1) is low-maintenance once integrated. |
| **Disadvantages** | The team owns security-critical session code (rotation, revocation, password reset) that a managed provider would otherwise own and continuously patch — non-trivial implementation *and* testing surface (mitigated by Section 10's testing stack covering this explicitly). |
| **Cost Impact** | $0 (vs. $0–$25+/month for managed auth depending on provider/MAU tier as the product scales). |
| **Learning Curve** | Medium — OAuth Authorization Code flow and JWT rotation are well-documented patterns, but require careful implementation; see Section 6.1 for the specific library corrections that reduce implementation risk here. |
| **Future Scalability** | If enterprise SSO/SAML becomes a requirement (Phase 3 RBAC/SSO, `12_Product_Requirements_Document.md` §15.3), a dedicated B2B-SSO provider (e.g., WorkOS, which specifically solves "add SSO to an existing custom-auth app" without replacing it) is the natural addition — not a replacement of this foundation. |

---

# 2. Backend Stack

## 2.1 API Framework — FastAPI

| Attribute | Detail |
|---|---|
| **Selected Technology** | FastAPI (Python 3.12), Uvicorn/Gunicorn for serving |
| **Purpose** | REST API for `preflight-api` — project/scan CRUD, job dispatch, report retrieval (`13_System_Architecture.md` §4–5). |
| **Why Selected** | Async-native — important because the API layer's hot paths (LLM Gateway calls, DB queries, S3 reads for transcripts) are I/O-bound; automatic OpenAPI 3.1 generation (`13_System_Architecture.md` §5) doubles as the source for a generated TypeScript API client (`openapi-typescript`), keeping frontend/backend types in sync without hand-maintained interfaces; native Pydantic v2 integration means the *same* schema classes validate API payloads, internal `preflight-core` data structures, and (per `14_AI_Architecture.md` §11.5) LLM structured outputs — "one model, many uses." Team already has FastAPI experience (`01_Project_Framework.md`). |
| **Alternatives Considered** | **Django REST Framework** — heavier, historically sync-first ORM (async support has improved but is not DRF's core strength), and DRF's batteries-included admin/auth/user model would conflict with the custom auth/RBAC model (Sections 1.5/6.1–6.2) — adopting DRF's user model and then overriding most of it is more work than starting lean. **Flask** — lacks native async, OpenAPI generation, and Pydantic integration; reaching FastAPI's baseline requires bolting on `flask-smorest`/`apispec`/`marshmallow`, at which point FastAPI is simply less code. **Node (Express/NestJS)** — would split the backend into a different language from `preflight-core` (Python), reintroducing exactly the cross-language integration tax ADR-004 (modular monolith) was designed to avoid. |
| **Advantages** | Async support for concurrent LLM Gateway/DB I/O; auto-generated OpenAPI → typed frontend client; shared Pydantic v2 schemas with `preflight-core`; mature ecosystem (`slowapi` for rate limiting, `13_System_Architecture.md` §5). |
| **Disadvantages** | Async discipline required — a blocking call inside an `async def` route (e.g., a synchronous DB driver call or a CPU-bound graph algorithm run inline) stalls the event loop for *all* concurrent requests; mitigated by offloading CPU-bound work to Celery (already the architecture, Section 2.2) and using `run_in_threadpool` for any unavoidable sync calls. |
| **Cost Impact** | $0. |
| **Learning Curve** | Low — team already has FastAPI experience. |
| **Future Scalability** | Stateless ASGI app behind an ALB — scales horizontally with zero code changes (`13_System_Architecture.md` §18); async model handles high-concurrency polling load (many users/CI jobs polling scan status simultaneously) efficiently. |

## 2.2 Task Queue — Celery + Redis

| Attribute | Detail |
|---|---|
| **Selected Technology** | Celery (task queue/worker framework) + Redis (broker, restated from `13_System_Architecture.md` §4/§9/§17) |
| **Purpose** | Asynchronous dispatch and execution of static-analysis and simulation jobs (FR-F4, NFR-4, NFR-7). |
| **Why Selected** | Celery is the de facto standard Python task queue with mature retry/idempotency primitives (`acks_late`, `task_reject_on_worker_lost` — directly serving NFR-7); supports the separate `STATICWORKER`/`SIMWORKER` queue routing the system architecture's component diagram specifies; Redis serves a dual role as both Celery broker *and* application cache (Section 5.2, `13_System_Architecture.md` §6.1/12.4) — one fewer managed service for a 3-person team. |
| **Alternatives Considered** | **RQ (Redis Queue)** — simpler API, but its retry/rate-limiting/multi-queue routing is less mature than Celery's, which matters directly for the per-scan budget-cap behavior (`13_System_Architecture.md` §9.2/§16.1: "simulations 8–10 are skipped, scan returns partial results"). **Dramatiq** — cleaner, more modern API than Celery, but a much smaller community — for a 3-person team that *will* hit Celery's well-known edge cases, the depth of existing Stack Overflow/GitHub-issue answers for Celery is itself a productivity asset. **AWS SQS + custom worker loop** — removes the Celery dependency but reimplements retry/dead-letter/concurrency-control logic Celery provides for free; SQS remains a reasonable Growth-phase *broker* swap (Celery supports SQS as a transport) if Redis-as-broker becomes a bottleneck (see Section 13, Risk #2). |
| **Advantages** | Mature ecosystem (Flower for queue monitoring, well-documented retry/backoff patterns); independent queue routing for `STATICWORKER` vs. `SIMWORKER` concurrency tuning; Redis broker doubling as cache reduces infrastructure surface at MVP. |
| **Disadvantages** | Celery's configuration surface is large and has well-known footguns (prefork-pool memory growth on long-lived workers, serialization gotchas with complex objects — mitigated by keeping task payloads to IDs/references, not full graph objects); using Redis as **both** broker and cache means a Redis outage on the single-EC2 MVP topology (ADR-006) simultaneously breaks job dispatch *and* the classifier cache (Section 13, Risk #2). |
| **Cost Impact** | $0 software cost. Redis is co-located on the single EC2 instance for MVP — $0 incremental; becomes ElastiCache `cache.t4g.micro` (~$15–30/month) in the Growth topology. |
| **Learning Curve** | Medium — Celery's routing/retry/result-backend configuration warrants a dedicated "Celery deep dive" early in Month 1–2, given how central it is to NFR-3 (cost caps) and NFR-7 (idempotency). |
| **Future Scalability** | Per `13_System_Architecture.md` §18: `STATICWORKER` and `SIMWORKER` scale independently as separate Fargate services in Growth, with Fargate Spot for the I/O-bound, interruption-tolerant `SIMWORKER`. If Redis's dual broker/cache role becomes a bottleneck, the roles split onto separate Redis instances (or ElastiCache for cache + Amazon MQ for broker) — a configuration change, not an application-code change, because the broker URL and cache client are already independently configured. |

## 2.3 Background Workers — Celery Worker Pool (Static + Simulation Queues)

| Attribute | Detail |
|---|---|
| **Selected Technology** | Two Celery worker pools: `worker-static` (CPU-bound, short tasks, default `prefork` pool) and `worker-sim` (I/O-bound, long tasks dominated by LLM latency, `gevent`/`eventlet`-style concurrent pool) |
| **Purpose** | Execute Module A/B (parse → graph → static risk scoring, NFR-1's <10s target) and Module C (sandboxed simulation, NFR-2's <2min target) pipelines asynchronously, per `13_System_Architecture.md`'s component diagram and `14_AI_Architecture.md` §9. |
| **Why Selected** | Separating queues is the load-bearing reason two pools exist: a burst of long-running simulation jobs must not starve the fast static-analysis jobs that power UC-1's "quick local scan" promise. Distinct concurrency models reflect the workloads — `worker-static` is CPU-bound and benefits from `prefork`'s process isolation; `worker-sim` is I/O-bound (waiting on LLM API latency, per `14_AI_Architecture.md` §13.1's token/latency model) and benefits from a concurrency model that doesn't pay one-OS-process-per-task overhead for mostly-idle-waiting tasks. |
| **Alternatives Considered** | **Single worker pool for everything** — simpler configuration, but directly causes the starvation problem above; rejected. **Separate microservices per queue** — rejected at the system-architecture level (ADR-004); the same modular-monolith reasoning applies to worker processes — they are separate *deployable units* (separate Celery worker invocations) within the same codebase, not separate services. |
| **Advantages** | Independent scaling/concurrency tuning per queue (small `--concurrency` for `worker-static`, higher for `worker-sim`); the queue separation done at MVP is precisely what makes the Growth-phase migration to independent Fargate services (`13_System_Architecture.md` §17.1) a deployment-config change rather than a redesign. |
| **Disadvantages** | Running `worker-sim` with a `gevent`/`eventlet` pool to get I/O concurrency without one-process-per-task overhead requires all I/O libraries in that code path (HTTP clients for LLM Gateway, the Docker SDK for sandbox lifecycle) to be monkey-patch-compatible — a known Celery+gevent integration risk class (Section 13, Risk #3) that should be piloted in Month 2–3, not discovered in Month 8. If gevent proves unstable, the documented fallback is `prefork` with a higher worker-replica count (more memory, less elegant, but well-understood). |
| **Cost Impact** | Included in EC2/Fargate compute cost — no separate line item. |
| **Learning Curve** | Medium — builds on Celery's general learning curve (Section 2.2); pool-type selection (`prefork` vs. `gevent`/`eventlet`) is an advanced-but-well-documented topic. |
| **Future Scalability** | Growth topology runs `worker-static` and `worker-sim` as independently auto-scaling Fargate services (`13_System_Architecture.md` §17.1/§18) — the MVP's queue separation is exactly the interface boundary that makes this a config change. |

## 2.4 Validation — Pydantic v2

| Attribute | Detail |
|---|---|
| **Selected Technology** | Pydantic v2 (Rust-core `pydantic-core`) |
| **Purpose** | Schema validation for API requests/responses, config-file parsing (FR-A1.4), and LLM structured outputs (`14_AI_Architecture.md` §3.2, §6.2–6.3, §8.2, §11.5). |
| **Why Selected** | Pydantic v2's Rust core is materially faster than v1 — directly relevant because validation sits in the hot path for *every* config upload (NFR-1: 1,000-node graphs in <10s) *and* every LLM structured-output parse (every classifier/attacker/victim/judge call, `14_AI_Architecture.md` §11.5); native FastAPI integration; `model_dump_json()` directly feeds the JSON report export (FR-D4.1). |
| **Alternatives Considered** | **Marshmallow** — older, slower, weaker static-type-checker integration (doesn't generate the same level of IDE/type-checking benefit). **`attrs` + `cattrs`** — lighter weight, but loses FastAPI's automatic request/response validation and OpenAPI generation, requiring a separate API-boundary validation layer — net more code for this stack. **Hand-written `dataclasses` + validators** — reinvents Pydantic with none of its tooling; pure maintenance liability. |
| **Advantages** | One schema definition reused across API contracts, internal `preflight-core` data models, *and* LLM output schemas — genuinely "one model, many uses," reducing the surface area for schema drift bugs. |
| **Disadvantages** | The Pydantic v1→v2 migration (2023–2024) broke many third-party libraries; the team must pin compatible versions of any Pydantic-dependent dependency and re-validate on upgrades — a known ecosystem-churn risk (Section 13, Risk #10), mitigated by `uv.lock` (Section 2.5). |
| **Cost Impact** | $0. |
| **Learning Curve** | Low — team has used Pydantic implicitly via FastAPI already; v2-specific idioms (`model_validate`, `model_dump`, `field_validator`) are a short adjustment. |
| **Future Scalability** | No scalability concern at this layer — per-call validation cost is dominated by I/O elsewhere in the pipeline. |

## 2.5 Dependency Management — `uv` (Python) + npm (Frontend) — **NEW DECISION (TECH-ADR-001/002)**

| Attribute | Detail |
|---|---|
| **Selected Technology** | **Python:** `uv` (Astral) with native workspace support across `preflight-core` / `preflight-api` / `preflight-worker` / `preflight-cli`. **Frontend:** npm (Node's bundled package manager) for the single `preflight-web` package. |
| **Purpose** | Reproducible, fast dependency resolution and installation across the monorepo layout defined in `13_System_Architecture.md` §4.1, and for `preflight-web`. |
| **Why Selected (Python — `uv`)** | `uv` became the dominant fast Python package/project manager through 2025–2026: a Rust implementation with a global content-addressable cache delivers 10–100× faster installs than `pip`/Poetry, and its native **workspace** concept maps directly onto the `core`/`api`/`worker`/`cli` layout — each becomes a workspace member depending on `preflight-core` via a path dependency, with one `uv.lock` for the whole monorepo. For a 3-person team running `uv sync` dozens of times a day during Module C development (`14_AI_Architecture.md` §14.3's "dozens of pipeline runs/day"), this speed is hours per week of compounding saved time over 15 months — not a cosmetic preference. `uv`'s CLI is deliberately `pip`-compatible (`uv pip install`), easing CI scripts and onboarding. |
| **Why Selected (Frontend — npm)** | `preflight-web` is presently a *single* JS package — `pnpm`'s headline advantage (a deduplicated, disk-efficient store shared across many packages in a monorepo) does not materialize for one package, and npm requires zero additional tooling beyond Node itself. |
| **Alternatives Considered** | **Poetry** — mature, widely adopted, excellent `pyproject.toml`-based resolution/lockfiles; the "safe, boring" choice. Rejected primarily on speed and on `uv`'s more first-class workspace support (Poetry's monorepo story relies on path dependencies or a community plugin, which is less integrated). **Decision note:** if `uv`'s relative immaturity causes a real blocker during the Month-1 pilot (Section 8), Poetry is the documented fallback — both use `pyproject.toml`, so the switching cost is low. **pip + pip-tools** — the "nothing fancy" baseline; rejected for lacking a true hash-pinned lockfile-by-default workflow and a monorepo workspace concept, both of which `uv` provides natively. **Conda** — irrelevant; no compiled-scientific-stack dependency justifies Conda's overhead, and per `14_AI_Architecture.md`'s correction, local model inference runs via Ollama (a separate binary), not via a Python-side PyTorch/CUDA stack that Conda would help manage. **pnpm / Yarn (frontend)** — `pnpm` becomes justified if a second JS package emerges (e.g., a shared TS-types package generated from the OpenAPI spec, Section 2.1) — tracked as a revisit trigger, not a current need. Yarn Berry's PnP mode has historically caused IDE/tooling friction not worth incurring for a single package. |
| **Advantages** | `uv`: dramatic install/resolve speed; single lockfile across the monorepo; `pip`-compatible CLI eases CI/onboarding. npm: zero setup, ships with Node. |
| **Disadvantages** | `uv` is comparatively new (first stable release 2024) — a smaller community knowledge base than `pip`/Poetry means occasional reliance on `uv`'s own docs/issue tracker rather than a quick Stack Overflow answer; some packages with unusual build backends have had `uv` compatibility issues (rapidly improving through 2025–2026, but warrants a Month-1 "does our actual dependency set install cleanly" spike — mirroring the Month-1-spike pattern `14_AI_Architecture.md` §6.7 established for the Attacker Agent). |
| **Cost Impact** | $0 (both free/open-source). |
| **Learning Curve** | Low — `uv`'s CLI is deliberately `pip`-like for anyone coming from `pip`/`pyproject.toml`; npm is near-zero. |
| **Future Scalability** | `uv` workspaces extend to additional Python packages (e.g., a future `preflight-sdk`) without restructuring. npm is adequate until/unless a JS monorepo need emerges (tracked above as a `pnpm` revisit trigger). |

---

# 3. AI/ML Stack

## 3.1 LLM Framework — LiteLLM + Custom `LLMGateway` (ADR-002 / AI-ADR Restated)

| Attribute | Detail |
|---|---|
| **Selected Technology** | LiteLLM (provider abstraction) wrapped by a thin custom `LLMGateway` class (`13_System_Architecture.md` §10, `14_AI_Architecture.md` §12) |
| **Purpose** | Single chokepoint for every LLM call (classifier, attacker, victim, judge — `14_AI_Architecture.md` §2), providing provider abstraction, cost accounting, caching, retries, and structured-output validation. |
| **Why Selected** | Restated from ADR-002: LiteLLM's `completion()` interface unifies Anthropic, OpenAI, and Ollama with minimal glue code and built-in per-call cost calculation — directly serving NFR-3/NFR-9 with far less custom code than bespoke per-provider adapters. The custom `LLMGateway` adds project-scoped budget enforcement, the classifier cache, and prompt-template versioning (`14_AI_Architecture.md` §12.1). |
| **Alternatives Considered** | **LangChain / LlamaIndex** — rejected: their agent/chain abstractions are heavyweight for what is fundamentally "render a templated prompt, call a model, validate structured output" — LiteLLM's thinner abstraction is materially easier to debug when a simulation produces an unexpected transcript, and LangChain's historically frequent breaking changes are a maintenance risk inconsistent with the "boring tech" philosophy. **Raw provider SDKs (`anthropic`, `openai` packages directly)** — rejected: the Victim Agent's "match whatever model the real deployment uses" requirement (`14_AI_Architecture.md` §7.4) structurally requires multi-provider routing; hand-rolling that abstraction reimplements what LiteLLM already provides. |
| **Advantages** | Provider-agnostic completion calls with normalized tool-calling across Anthropic/OpenAI/Ollama; built-in cost tracking feeds NFR-3's per-scan budget cap directly; one circuit-breaker/fallback layer (`14_AI_Architecture.md` §12.3) covers all four AI roles. |
| **Disadvantages** | LiteLLM itself is a moving target (frequent releases tracking new provider features/models) — version pinning plus a small conformance test suite (one fixture per supported provider, per `14_AI_Architecture.md` §7.7) is required to catch tool-call-format drift across LiteLLM/provider updates (Section 13, Risk #6). |
| **Cost Impact** | $0 software cost; LLM API spend tracked separately (Section 12). |
| **Learning Curve** | Low–Medium — LiteLLM's API mirrors the OpenAI `chat.completions` shape closely, which most LLM-experienced engineers already know; the `LLMGateway` wrapper's caching/budget logic is custom and team-specific, requiring onboarding documentation. |
| **Future Scalability** | Adding a new provider (e.g., Google Gemini, Section 3.3) is a LiteLLM model-string change plus a `model_routing.yaml` entry (`13_System_Architecture.md` §10.2) — no architectural change. |

## 3.2 Local Models — Ollama + 1–4B Models (AI-ADR-002 Restated, Concrete Picks)

| Attribute | Detail |
|---|---|
| **Selected Technology** | Ollama (serving layer) running **Qwen2.5-3B-Instruct** (Q4_K_M) as the primary local model for the Tool Classifier's primary tier, with **Qwen2.5-1.5B-Instruct** as a faster fallback for high-volume/latency-sensitive batch classification |
| **Purpose** | Local inference for the Tool Classifier's primary path (`14_AI_Architecture.md` §3.3, §13) and for pipeline/orchestration testing of Sections 6–9's state machine where attack *quality* is not under test. |
| **Why Selected** | Per AI-ADR-002's correction: the RTX 2050's **4GB VRAM** (not 24GB — that's system RAM) rules out usable-throughput 7B+ models locally. Qwen2.5-3B-Instruct at Q4_K_M quantization (~1.8–2.3GB) fits comfortably with headroom for KV cache at the short context lengths typical of tool-description classification (`14_AI_Architecture.md` §3.2's `ToolClassificationRequest` is a few hundred tokens), running at 15–35 tok/s on-GPU — fast enough for interactive `preflight scan` runs. Qwen2.5-1.5B-Instruct (~1.0GB) provides a lower-latency option if classification throughput becomes a bottleneck during a large-config scan. |
| **Alternatives Considered (models)** | **Phi-3.5-mini-instruct (3.8B)** — strong instruction-following, slightly larger than the Qwen2.5-3B pick; a reasonable A/B candidate. **Llama-3.2-3B-Instruct** — solid baseline, Meta's license terms are acceptable for this commercial use. Per `14_AI_Architecture.md` §14.4, **the team should re-survey Ollama's model library at Month 1–2** rather than treating these as fixed — the *strategy* (1–4B local for classification) is durable; the specific checkpoint is not. |
| **Alternatives Considered (serving layer)** | **`llama.cpp` directly** — Ollama is built on `llama.cpp` and adds model pull/serve/REST-API management; using `llama.cpp` directly means reimplementing that management layer for no benefit. **LM Studio** — GUI-focused, less scriptable/CI-friendly than Ollama's CLI + REST API. **vLLM** — designed for datacenter-GPU multi-request batching/throughput; its value proposition (continuous batching across many concurrent requests on A100/H100-class hardware) is irrelevant to a single-developer 4GB-VRAM laptop and partially incompatible with it. |
| **Advantages** | Zero marginal inference cost for the highest-call-volume role (every tool in every scan, before caching); CPU-friendly fallback if GPU contention occurs (Ollama gracefully offloads layers to CPU, just slower). |
| **Disadvantages** | Throughput at the 1.5–3.8B tier is "usable for interactive dev," not "fast" — large configs (hundreds of tools, before cache warm-up) may take tens of seconds for classification alone on a cold cache; mitigated by the classifier cache (`13_System_Architecture.md` §11.3) which is the actual primary cost/latency control, with the local model as the *uncached-call* fallback. |
| **Cost Impact** | $0 (runs on existing dev hardware). |
| **Learning Curve** | Low — `ollama pull <model>` + `ollama serve`, consumed via LiteLLM's Ollama provider. |
| **Future Scalability** | Phase 2 distillation (AI-ADR-003, `14_AI_Architecture.md` §15.2) targets exactly this 1.5–3B tier as the *inference* target after a cloud-rented QLoRA fine-tune — the local-serving choice made here is also the Phase 2 production-inference target, not a throwaway dev convenience. |

## 3.3 Hosted Models — Anthropic (Primary) + OpenAI (LiteLLM Fallback)

| Attribute | Detail |
|---|---|
| **Selected Technology** | `claude-sonnet-4-6` for Attacker, default Victim, and Judge (Tier 2) roles; `claude-haiku-4-5` for Tool Classifier escalation (`14_AI_Architecture.md` §13); OpenAI models available via LiteLLM as a secondary/fallback provider and as a configurable Victim-model option. |
| **Purpose** | Quality-sensitive simulation roles (Section 3.1's "the three things LLMs are genuinely needed for") and classifier escalation for low-confidence cases. |
| **Why Selected** | Per `14_AI_Architecture.md` §6.4, the Attacker and Judge (Tier 2) roles are explicitly **not** the place to economize on model quality — a weak attacker produces false negatives that undermine PreFlight's core value proposition (a security scanner that misses real vulnerabilities is worse than no scanner). `claude-haiku-4-5` for classifier escalation balances the 5–15% escalation rate's cost against accuracy (`14_AI_Architecture.md` §3.3). **Note on model naming:** model identifiers evolve rapidly; the team should treat `model_routing.yaml` (`13_System_Architecture.md` §10.2) as the single source of truth and re-verify current model names/pricing at implementation time and periodically thereafter — the *routing strategy* (cheap-tier for classifier escalation, quality-tier for Attacker/Judge, user-configurable for Victim) is the durable decision. |
| **Alternatives Considered** | **Google Gemini (via LiteLLM)** — viable second fallback, cost-competitive; deferred as a *primary* alternative because the team has no prior experience with Gemini's tool-calling format quirks, and OpenAI is chosen as the secondary provider for two additional reasons: (a) much of the academic red-teaming literature (`14_AI_Architecture.md` §16.1's corpora) benchmarks against GPT-family models, giving better comparability for the research deliverable (G4); (b) LiteLLM's normalization of OpenAI's and Anthropic's tool-calling formats is its most battle-tested path. **Single-provider (Anthropic-only)** — considered for simplicity, but rejected: `14_AI_Architecture.md` §7.4 requires the Victim Agent to match "whatever the user is testing," which structurally requires multi-provider support regardless of which provider is "primary" for the Attacker/Judge. |
| **Advantages** | Frontier-model quality where it matters most (Attacker/Judge — directly determines false-negative rate, the PRD §12 KPI); multi-provider Victim support is a genuine product feature, not just an engineering nicety. |
| **Disadvantages** | Recurring per-call cost (Section 12); provider-side rate limits/outages affect the simulation pipeline — mitigated by the circuit-breaker/fallback design (`14_AI_Architecture.md` §12.3). |
| **Cost Impact** | The dominant variable cost in the system. Development-phase: $50–100/month (`14_AI_Architecture.md` §14.3). Production per-scan: $0.10–$2.00/scan target (NFR-3, `14_AI_Architecture.md` §13.1). |
| **Learning Curve** | Low — both Anthropic's and OpenAI's APIs are widely documented; LiteLLM normalizes the differences. |
| **Future Scalability** | Adding/swapping providers is a `model_routing.yaml` change. If the data moat (`11_Final_Startup_Selection.md` §11) eventually justifies preference-tuning a custom Attacker model (Phase 3, explicitly deferred per `14_AI_Architecture.md` §6.5), that model slots into the same LiteLLM/`LLMGateway` abstraction as just another routable model. |

## 3.4 Embedding Models — sentence-transformers `all-MiniLM-L6-v2` (AI-ADR-005 Restated)

| Attribute | Detail |
|---|---|
| **Selected Technology** | `sentence-transformers/all-MiniLM-L6-v2` (22M parameters, 384-dim, CPU inference) |
| **Purpose** | Resource-name deduplication (`14_AI_Architecture.md` §4.2), RAG technique retrieval (§16.4), and CVE-signature matching (§16.2). |
| **Why Selected** | Restated from `14_AI_Architecture.md` §17.1: every embedding use case here operates on **short technical text** (tool/resource names, CVE descriptions, technique snippets), where this model's accuracy is more than sufficient, and its CPU-only footprint imposes zero contention with the RTX 2050's scarce 4GB VRAM (already committed to the local classifier, Section 3.2). |
| **Alternatives Considered** | **Hosted embedding APIs (OpenAI/Anthropic/Voyage)** — rejected: adds per-call cost and network latency to a high-volume, low-stakes, every-scan operation, and couples a core pipeline step to API availability for no accuracy benefit at this text length. **`all-mpnet-base-v2`** — better accuracy on longer/more semantically complex text, but ~4× slower; unjustified per `14_AI_Architecture.md` §17.1's reasoning for short strings. **`bge-small-en-v1.5`** — a competitive same-class alternative, slightly larger than MiniLM-L6; worth a quick benchmark during the Month 2–3 corpus-building work (`14_AI_Architecture.md` §21.1), but MiniLM-L6's speed and zero-VRAM-contention property make it the safe default. |
| **Advantages** | Free, fast (single-digit ms/sentence on CPU), zero GPU contention, mature/stable model (unlikely to be deprecated, unlike fast-moving local-LLM checkpoints in Section 3.2). |
| **Disadvantages** | None material at this scale; if corpora grow into domains requiring longer-context or multilingual embeddings (not anticipated for PreFlight's English-language technical-text use cases), a model swap would be a config change to the embedding step, isolated from the FAISS/storage layer (Section 4.3). |
| **Cost Impact** | $0. |
| **Learning Curve** | Low — `sentence-transformers` has a two-line usage API. |
| **Future Scalability** | FAISS-in-process (Section 4.3) scales to low-thousands of vectors comfortably; the embedding model choice is independent of, and does not block, the Phase 2 `pgvector` revisit trigger. |

## 3.5 Evaluation Frameworks — Custom pytest Harness (DeepEval/promptfoo Considered, Deferred) — **NEW DECISION (TECH-ADR-012)**

| Attribute | Detail |
|---|---|
| **Selected Technology** | A custom evaluation harness built on `pytest` (+ `pytest-xdist` for parallelism), implementing the three suites specified in `14_AI_Architecture.md` §20 (Classifier Eval, Known-CVE Detection, Simulation Calibration). `promptfoo` is permitted as an **informal developer tool** during prompt iteration (Section 10), but is not a CI dependency. |
| **Purpose** | CI-gated evaluation of the AI subsystem against the PRD §12 KPIs — most critically, recall@top-10 on the Known-CVE Detection Suite (≥80% target). |
| **Why Selected** | The evals defined in `14_AI_Architecture.md` §20 are **domain-specific**: "did the pattern-detector + simulation pipeline surface this CVE's attack path in the top-10 ranked findings" is not a metric any general-purpose LLM-eval framework ships out of the box. Generic frameworks (DeepEval, promptfoo, Ragas) are built around prompt→response pairs and standard NLP metrics (relevance, faithfulness, hallucination) that don't map onto a graph-conditioned, multi-stage pipeline's pass/fail criteria. A thin `pytest` harness — parametrized test cases over the CVE corpus (`14_AI_Architecture.md` §21.2) — integrates *directly* with the existing CI gating split (per-PR vs. nightly, §20.1/§20.2) without adding a second test-runner/config layer. |
| **Alternatives Considered** | **DeepEval** — its assertion helpers (e.g., structured-output field-match assertions) are a reasonable fit for the **Classifier Eval Suite** specifically (§20.1's "does output match expected field values"); adopting it for *only one of three suites* was considered and rejected for consistency — three suites, one harness, is simpler to maintain than two harnesses for three suites. **promptfoo** — genuinely useful as a **developer-facing** tool for iterating on the Jinja2 prompt templates (`14_AI_Architecture.md` §11) — fast, CLI-driven prompt A/B comparison. Recommendation: use it informally during prompt development, but do not make it a CI dependency (avoids a second eval-config format alongside the `pytest` harness). **Ragas** — RAG-specific metrics (faithfulness, answer relevancy) target full RAG-QA pipelines; PreFlight's retrieval step (§16) is a simple top-k FAISS lookup feeding a structured prompt, not a generative QA pipeline Ragas's metrics were designed to evaluate. |
| **Advantages** | Zero new framework dependency; the recall@top-10 metric is defined *exactly* as the PRD §12 KPI requires (no translation layer between "what the framework measures" and "what the contract says"); runs via the same `pytest`/CI infrastructure as everything else (Section 10). |
| **Disadvantages** | The team builds its own eval-result reporting (a DeepEval/promptfoo dashboard would provide this "for free") — mitigated by writing eval results to a JSON/CSV artifact uploaded as a GitHub Actions artifact (viewable in the Actions UI with zero extra infrastructure); if eval-result trend-tracking across prompt-template versions (`14_AI_Architecture.md` §11.1) becomes valuable, logging eval runs to a Postgres table (reusing existing infra, Section 5.1) is preferred over adopting a heavyweight MLOps eval platform (e.g., Weights & Biases) given the cost/ops overhead for a 3-person team. |
| **Cost Impact** | $0. |
| **Learning Curve** | Low — builds directly on `pytest` (Section 10.1). |
| **Future Scalability** | `pytest -n auto` (via `pytest-xdist`) parallelizes eval-suite growth (more CVE configs, more classifier examples) without runtime blowup; the harness's design (parametrized fixtures over corpora) scales linearly with corpus size, which is the expected growth mode per `14_AI_Architecture.md` §21. |

---

# 4. Graph Analysis Stack

## 4.1 Graph Library — NetworkX (ADR-001 Restated)

| Attribute | Detail |
|---|---|
| **Selected Technology** | NetworkX (`DiGraph`), in-memory, with node-link JSON serialization into Postgres JSONB |
| **Purpose** | The `PermissionGraph` data structure (FR-A2.1–A2.5) — construction, traversal, path-finding, serialization for the frontend (Cytoscape) and storage. |
| **Why Selected** | Restated from ADR-001: at the NFR-1 scale ceiling (~1,000 nodes / ~5,000 edges), NetworkX's path-finding (Yen's k-shortest-paths, Dijkstra) completes in milliseconds-to-low-seconds — comfortably within budget. JSONB-in-Postgres avoids operating a second database system, and Cytoscape.js consumes the same node-link JSON directly with no transformation layer. |
| **Alternatives Considered** | **Neo4j / Amazon Neptune** — explicitly rejected for MVP per ADR-001 (not justified at this scale; revisit trigger documented in Section 4.3 below). **igraph** — C-backed, faster on very large graphs, but (a) NFR-1's bound already makes NetworkX's performance adequate (`14_AI_Architecture.md` §5.2's complexity analysis: ~1.5×10⁸ operations for K=10 Yen's, well within 10 seconds), and (b) igraph's Python API is less idiomatic and its installation has had platform-specific build friction historically — a real cost for a 3-person team setting up dev environments across different OSes. |
| **Advantages** | Pure-Python (partially C-backed) — no native build complexity for the team's dev setup; rich algorithm library covers every Module B requirement (Yen's, Dijkstra, BFS/DFS reachability) without custom implementations. |
| **Disadvantages** | Pure in-memory — each scan's graph computation is single-process/single-machine, which is fine at NFR-1's scale but is the reason cross-project graph queries (Phase 2 MSSP feature, UC-8) require the strategy in Section 4.3 rather than "just query NetworkX across projects." |
| **Cost Impact** | $0. |
| **Learning Curve** | Low — team's existing Python/data-science background (`01_Project_Framework.md`: NumPy/Pandas/Scikit-Learn) transfers directly to NetworkX's API conventions. |
| **Future Scalability** | Per-project in-memory computation with depth limits and top-N simulation selection (FR-B2.3) bounds complexity regardless of overall organization size (`13_System_Architecture.md` §18). The `PermissionGraph` abstraction in `core/graph/` is designed (repository pattern) so its backing store can be swapped without changing callers — see Section 4.3. |

## 4.2 Graph Algorithms — Yen's k-Shortest Paths, BFS/DFS Reachability, Custom Pattern Detectors

| Attribute | Detail |
|---|---|
| **Selected Technology** | `nx.shortest_simple_paths` (NetworkX's Yen's-algorithm implementation, weighted) for the Generic Path Detector; `nx.bfs_tree`/`nx.descendants` for bounded-depth reachability (Lethal Trifecta's `bfs_reachable`); hand-written `PatternDetector` plugins (`13_System_Architecture.md` §12.2) for Confused Deputy / Credential Reuse / Lethal Trifecta (`14_AI_Architecture.md` §5.3–5.5). |
| **Purpose** | Implements FR-B2.1–B2.4 — discovering and ranking candidate attack paths fed to the simulation engine. |
| **Why Selected** | Restated/expanded from `14_AI_Architecture.md` §5: graph traversal is "exact graph traversal" (Section 1.2's "what is not AI" principle) — using NetworkX's tested, documented implementations rather than hand-rolling them eliminates an entire class of subtle correctness bugs (e.g., off-by-one depth limits, incorrect handling of negative-weight edge cases — though the `edge_cost` transformation in §5.1 specifically avoids negative weights to keep Dijkstra/Yen's applicable). |
| **Alternatives Considered** | **Hand-implemented Yen's algorithm** — rejected outright; NetworkX's implementation is correct, tested by a large user base, and reinventing it is pure risk for zero benefit. **igraph's path algorithms** — same library-level alternatives discussion as Section 4.1; not separately re-litigated here. |
| **Advantages** | The custom `PatternDetector` plugin registry (`13_System_Architecture.md` §12.2 / `14_AI_Architecture.md` §5.3–5.5) is the right place for PreFlight-specific domain logic (Lethal Trifecta's ordered-reachability check), while the underlying primitive operations (shortest paths, BFS) are NetworkX's battle-tested code — a clean division between "library does the generic graph math" and "we encode the security-domain knowledge." |
| **Disadvantages** | The adaptive `k` reduction for dense graphs (`14_AI_Architecture.md` §5.2: `k = max(2, 10 // num_agent_sink_pairs)`) is a hand-written heuristic without NetworkX support — requires its own unit tests and is a documented Phase 1 implementation detail, not an afterthought. |
| **Cost Impact** | $0. |
| **Learning Curve** | Low–Medium — NetworkX's path-finding API is straightforward; the `PatternDetector` protocol (`13_System_Architecture.md` §12.2) requires understanding the domain semantics (what *is* a Confused Deputy in graph terms) more than the library API. |
| **Future Scalability** | New pattern detectors (Phase 2/3) register via the same `PatternDetector` protocol without touching the path-finding core — NFR-10's "extensibility" requirement is satisfied by this plugin design. |

## 4.3 Future Graph Database Strategy — Decision Tree (ADR-001 Extension) — **NEW (TECH-ADR-013)**

| Attribute | Detail |
|---|---|
| **Selected Technology (now)** | NetworkX in-memory + Postgres JSONB (Section 4.1, ADR-001) — **no change for MVP.** |
| **Purpose** | Document the *staged* revisit path for graph storage as cross-project/cross-org query needs emerge (Phase 2 MSSP feature, UC-8; `13_System_Architecture.md` §8.1's revisit trigger). |
| **Why This Staging** | ADR-001's revisit trigger ("cross-project graph queries... e.g., 'show me every agent across the org that can reach this specific credential'") does not necessarily require jumping straight to a dedicated graph database. **Stage 1 (now):** NetworkX per-scan, JSONB snapshots. **Stage 2 (if the trigger fires at moderate scale):** PostgreSQL `WITH RECURSIVE` queries over a normalized `nodes`/`edges` table (or directly over `graph_snapshots.graph_json` via JSONB path operators for simpler cases) — Postgres can express bounded-depth graph traversal without a new database system, continuing the pattern established by ADR-001/ADR-005/AI-ADR-005 ("avoid a second data-store system"). **Stage 3 (only if Stage 2 measurably fails at true enterprise/cross-org scale):** Neo4j (self-managed — operationally heavy for a small team) or Amazon Neptune (managed, but Neptune Serverless pricing starts around $0.10/NCU-hour with a non-trivial practical minimum — a real recurring cost). |
| **Alternatives Considered** | **Jump directly to Neo4j when the ADR-001 trigger fires** — rejected as the *default* path; Stage 2 (recursive CTEs) is materially cheaper (zero new infrastructure) and likely sufficient for the UC-8 MSSP use case at the scale a 3-person team's customer base will realistically reach within 15–24 months. **`pgvector`** — solves a different problem (vector similarity search, already addressed in Section 3.4/4.3 for embeddings) and is not a substitute for graph traversal — listed here only to disambiguate it from the graph-database question. |
| **Advantages** | Defers a real operational-overhead decision until there's evidence it's needed; Stage 2 is a SQL-only change (no new service, no new credentials in Secrets Manager, no new backup/monitoring surface). |
| **Disadvantages** | Recursive CTEs over JSONB are less ergonomic than Cypher for genuinely graph-shaped queries — if Stage 2 is reached, expect some query-authoring friction (Postgres recursive CTEs over JSON are verbose) relative to a native graph-query language; this friction is the actual *signal* that Stage 3 may be warranted. |
| **Cost Impact** | $0 at Stage 1 and Stage 2 (existing Postgres instance). Stage 3: Neo4j self-hosted adds an EC2/ECS line item + ops burden; Neptune Serverless adds a metered AWS line item. |
| **Learning Curve** | Stage 2: Low–Medium for a team already comfortable with SQL (`01_Project_Framework.md` lists SQL as an existing skill) — recursive CTEs are a well-documented Postgres feature. Stage 3: Medium–High (Cypher is a new query language). |
| **Future Scalability** | This staged approach is itself the scalability strategy — each stage is adopted only when the previous stage's limits are empirically reached, avoiding both premature complexity *and* a painful late migration (the `PermissionGraph` repository-pattern abstraction noted in Section 4.1 is what makes each stage transition a backing-store swap rather than an application rewrite). |

---

# 5. Database Stack

## 5.1 Relational Database — PostgreSQL 15+ (RDS, Restated from `13_System_Architecture.md` §7)

| Attribute | Detail |
|---|---|
| **Selected Technology** | PostgreSQL 15+ on Amazon RDS (`db.t4g.micro` for MVP), SQLAlchemy 2.0 ORM + Alembic migrations |
| **Purpose** | Durable storage for projects, configs, scans, findings, simulations, access control (`13_System_Architecture.md` §7.1's ERD). |
| **Why Selected** | JSONB support is load-bearing — `graph_snapshots.graph_json` and `simulations.transcript_ref`/metadata rely on JSONB's flexibility *and* its GIN-indexability (`13_System_Architecture.md` §7.2). Relational integrity for projects/findings/access-control plus JSONB flexibility for semi-structured graph/transcript data means one system covers both needs (ADR-005). |
| **Alternatives Considered** | **MySQL** — weaker JSON support and indexing maturity relative to Postgres's JSONB+GIN, which is directly used by the graph-storage design (Section 4) and the future drift-detection diffing (`13_System_Architecture.md` §8.2). **MongoDB (+ separate relational store)** — rejected per ADR-005: splitting "relational" data (users, projects, access control) and "document" data (graphs, transcripts) across two databases is exactly the second-system operational tax this project's ADRs consistently avoid; JSONB-in-Postgres covers the document-shaped data adequately at this scale. |
| **Advantages** | One database for everything (relational + semi-structured); RDS-managed backups/encryption-at-rest (NFR-6); SQLAlchemy 2.0 + Alembic is a mature, well-documented ORM/migration combination the team's SQL skills (`01_Project_Framework.md`) transfer to directly. |
| **Disadvantages** | `db.t4g.micro` is a real ceiling — adequate for MVP traffic but the first infrastructure component likely to need vertical scaling as design-partner usage (G2) grows; this is an expected, budgeted scaling step (Section 12), not a design flaw. |
| **Cost Impact** | ~$13–15/month (`db.t4g.micro`, on-demand, ap-south-1) + ~$2–3/month storage (20GB gp3) at MVP. Growth: vertical scale-up, then a read replica for reporting queries (`13_System_Architecture.md` §18). |
| **Learning Curve** | Low — team has SQL experience; SQLAlchemy 2.0's newer async-capable API has a moderate learning curve if async DB access is adopted (optional at MVP — sync SQLAlchemy via FastAPI's threadpool is acceptable initially). |
| **Future Scalability** | Vertical scale-up → read replica (reporting queries) → (if ever needed) Postgres-compatible Aurora for storage-layer elasticity — all standard RDS migration paths with no application-layer rewrite required. |

## 5.2 Caching Layer — Redis (Co-located MVP → ElastiCache Growth)

| Attribute | Detail |
|---|---|
| **Selected Technology** | Redis — co-located on the single EC2 instance for MVP (ADR-006), ElastiCache `cache.t4g.micro` in Growth |
| **Purpose** | Classifier-result read-through cache (`13_System_Architecture.md` §11.3, `14_AI_Architecture.md` §12.4), Celery broker (Section 2.2), and `slowapi` rate-limit token buckets (`13_System_Architecture.md` §5). |
| **Why Selected** | Redis's data structures (sorted sets for rate limiting, simple key-value with TTL for the classifier cache) cover all three use cases with one technology; already a hard requirement as the Celery broker (Section 2.2), so using it as the cache too is "free" infrastructure reuse rather than a separate decision. |
| **Alternatives Considered** | **Memcached** — simpler, but lacks the data structures needed for rate limiting (sorted sets) — would require Redis *anyway* for Celery, making Memcached a redundant second cache technology for marginal benefit. **In-process LRU cache (`cachetools`)** — inadequate once multiple API/worker processes need a *shared* cache; the classifier cache (`13_System_Architecture.md` §11.3) must be visible across all workers and API instances, which an in-process cache cannot provide. |
| **Advantages** | One technology serves cache + broker + rate-limiting; ElastiCache's managed failover in Growth directly addresses the single-EC2 MVP's Redis-as-SPOF risk (Section 13, Risk #2). |
| **Disadvantages** | At MVP, Redis is co-located with the API/worker containers on one EC2 instance — a Redis crash affects caching, job dispatch, *and* rate limiting simultaneously (documented, accepted MVP risk per Section 13). |
| **Cost Impact** | $0 incremental at MVP (co-located); ~$15–30/month for ElastiCache `cache.t4g.micro` in Growth. |
| **Learning Curve** | Low — Redis's key-value/TTL model is simple; the classifier cache's key design (`hash(server_id + tool_name + description + schema)`) is the only non-trivial design decision, and it's already specified in `13_System_Architecture.md` §11.3. |
| **Future Scalability** | ElastiCache with Multi-AZ replication in Growth resolves the SPOF risk; if cache and broker roles ever need independent scaling, they split onto separate Redis/ElastiCache instances — a configuration change (separate connection URLs already exist in the codebase by design). |

## 5.3 Object Storage — Amazon S3 (Restated from `13_System_Architecture.md` §7.3/§17/§19.2)

| Attribute | Detail |
|---|---|
| **Selected Technology** | Amazon S3 — buckets for simulation transcripts, PDF/JSON reports, and `preflight-web`'s static build (served via CloudFront) |
| **Purpose** | Large-object storage where Postgres would be a poor fit (full LLM simulation transcripts, which can span many turns — `13_System_Architecture.md` §7.3), plus static frontend hosting. |
| **Why Selected** | `simulations.transcript_ref` stores only an S3 key, keeping Postgres rows small and queryable; S3 lifecycle policies (transition to Infrequent Access after 30 days, Glacier after 90 — `13_System_Architecture.md` §19.2) provide cost optimization "for free" via configuration, not code. |
| **Alternatives Considered** | **Store transcripts in Postgres (JSONB or TEXT columns)** — rejected per `13_System_Architecture.md` §7.3: bloats the primary database, complicates backups, and defeats the "Postgres holds pointers + summary fields for fast querying" design. **Self-hosted object store (MinIO)** — adds operational overhead (a stateful service to back up, monitor, and secure) with no benefit at this scale, and reimplements S3's lifecycle-policy cost optimization that comes free with S3. |
| **Advantages** | Effectively infinite capacity with no operational management; lifecycle policies are declarative (Terraform-managed, Section 9.3); SSE-KMS encryption at rest (NFR-6) is a bucket-policy setting, not custom code. |
| **Disadvantages** | None material — S3 is the "boring, correct" choice for this workload. |
| **Cost Impact** | <$1–2/month at MVP transcript/report volume; scales with simulation volume but remains a small line item even at meaningful usage due to per-GB pricing and lifecycle transitions. |
| **Learning Curve** | Low — team has AWS Fundamentals (`01_Project_Framework.md`); S3 SDK usage (`boto3`) is straightforward. |
| **Future Scalability** | No scaling concern — S3 scales transparently; the only future consideration is the research-export scrubbing pipeline (FR-D4.2, `14_AI_Architecture.md` §21.4) running *before* any external-bucket/public-release step, which is a process control, not an S3 capability gap. |

---

# 6. Security Stack

## 6.1 Authentication — PyJWT + argon2-cffi (Direct) + Authlib — **CORRECTION (TECH-ADR-003/004/005)**

| Attribute | Detail |
|---|---|
| **Selected Technology** | **JWT:** PyJWT (replacing `13_System_Architecture.md` §6's `python-jose`). **Password hashing:** `argon2-cffi` used directly (replacing `13_System_Architecture.md` §6's "Argon2 via `passlib`"). **OAuth:** Authlib (GitHub OAuth, retained from `13_System_Architecture.md` §6.2). **Framework:** a lean custom implementation (replacing `13_System_Architecture.md` §6's "`fastapi-users` or custom JWT" either/or). |
| **Purpose** | Token issuance/verification for the session model (Section 1.5), password storage, GitHub OAuth login. |
| **Why Selected — Correction 1 (python-jose → PyJWT)** | `python-jose` has a documented history of **algorithm-confusion vulnerabilities** in JWT verification (where, under certain misconfigurations, a token's `alg` header can be manipulated to bypass signature verification — a well-known JWT library vulnerability class) and has seen markedly less recent maintenance activity than PyJWT. PyJWT is the library most current FastAPI security guidance recommends, has a smaller/simpler API surface (reducing misconfiguration risk — e.g., requiring the verifier to explicitly specify allowed algorithms), and is actively maintained. For a *security product*, shipping a JWT library with an algorithm-confusion CVE history in its own auth layer is a credibility risk that a security researcher reviewing PreFlight (a near-certainty given its positioning) would flag immediately. |
| **Why Selected — Correction 2 (passlib → argon2-cffi direct)** | `passlib` has had **no release since 2020** and has a widely-reported compatibility break with `bcrypt` >= 4.0 (passlib's bcrypt backend depends on an internal `bcrypt` attribute that was removed). Since Argon2 (OWASP's recommended algorithm for new applications) was already the chosen hashing algorithm, there is no reason to route through `passlib`'s multi-algorithm abstraction layer at all — `argon2-cffi` (the reference Python binding to the Argon2 reference implementation) is used directly, removing an unmaintained dependency from the single most security-critical code path in the application. |
| **Why Selected — Correction 3 (fastapi-users → custom)** | `fastapi-users` provides a complete, opinionated user/auth subsystem — valuable for greenfield CRUD apps with a standard user model. PreFlight's data model requires `organizations`/`memberships`/project-role tables **from day one** (`13_System_Architecture.md` §6.1, ADR-006) for reasons specific to PreFlight's domain (multi-tenant-ready schema without a later migration). Bending `fastapi-users`'s user model to fit this custom org/role schema typically costs *more* integration effort than writing the ~300–500 lines of custom auth code (login, refresh, OAuth callback, `current_user`/`current_membership` FastAPI dependencies) directly with PyJWT + Authlib + `argon2-cffi`. The custom code is also fully legible to a 3-person team that must be able to explain and defend every line of its own auth implementation — itself a credibility asset for a security product. |
| **Alternatives Considered** | Managed auth platforms (Auth0/Clerk/WorkOS) — discussed and deferred in Section 1.5, same conclusion applies. `fastapi-users` — discussed above. For OAuth specifically, Authlib has no strong FastAPI-ecosystem competitor and is retained unchanged from `13_System_Architecture.md` §6.2. |
| **Advantages** | PyJWT and `argon2-cffi` are both minimal, single-purpose, actively-maintained libraries with small attack surfaces — directly aligned with both the "boring tech" philosophy *and* PreFlight's own security-product credibility narrative (the same narrative that motivates Section 6.4's vulnerability-scanning choices). |
| **Disadvantages** | Slightly more boilerplate than `fastapi-users` for the login/refresh/OAuth-callback routes — estimated at **1–2 extra developer-days**, a clearly worthwhile trade given the reasons above. |
| **Cost Impact** | $0. |
| **Learning Curve** | Low — PyJWT and `argon2-cffi` both have minimal, well-documented APIs (arguably *simpler* than `passlib`'s multi-algorithm abstraction, since there's only one algorithm to configure). |
| **Future Scalability** | No scalability concern. If enterprise SSO/SAML is needed later (Phase 3, `12_Product_Requirements_Document.md` §15.3), it's *added alongside* this foundation via a dedicated B2B-SSO provider (e.g., WorkOS) — not a replacement, consistent with Section 1.5. |

## 6.2 Authorization — Application-Layer RBAC (ADR-009 Restated)

| Attribute | Detail |
|---|---|
| **Selected Technology** | Application-layer project-membership/role checks (Owner/Editor/Viewer per `13_System_Architecture.md` §6.1), implemented as FastAPI dependencies |
| **Purpose** | Enforce who can view/edit/trigger scans per project. |
| **Why Selected** | Restated from ADR-009: simpler to implement, test, and reason about for a 3-person team than Postgres Row-Level Security (RLS) at MVP, while the `organizations`/`memberships` schema (present from day one per ADR-006) means the *data model* is already RLS-ready if/when RLS becomes worthwhile. |
| **Alternatives Considered** | **Postgres RLS** — deferred per ADR-009, revisit trigger: direct external DB access (e.g., a future BI tool connecting straight to Postgres) — at that point, app-layer checks alone are insufficient because a non-application client could bypass them. **Dedicated policy engine (OPA/Open Policy Agent, Oso)** — rejected as operational overkill for a 3-role model; becomes worth revisiting in Phase 3 if the role model grows substantially more complex (e.g., per-resource ABAC for MSSP multi-client isolation, UC-8). |
| **Advantages** | Authorization logic lives in the same codebase/language as the business logic it protects — easy to test with `pytest` (Section 10), easy for a 3-person team to audit. |
| **Disadvantages** | Any *new* direct-DB-access surface (a future internal admin tool, a BI connector) must independently re-implement these checks or be denied direct access entirely — a discipline requirement, not an automatic guarantee (this is exactly RLS's advantage, and exactly why RLS is the documented Phase 3 fallback). |
| **Cost Impact** | $0. |
| **Learning Curve** | Low — straightforward FastAPI dependency-injection pattern. |
| **Future Scalability** | The `organizations`/`memberships` schema being present from day one (ADR-006) is precisely what makes a future RLS migration additive rather than a schema rewrite. |

## 6.3 Secrets Management — AWS Secrets Manager (Restated from `13_System_Architecture.md` §6.1/§15.3)

| Attribute | Detail |
|---|---|
| **Selected Technology** | AWS Secrets Manager, injected via EC2 instance profile (MVP) / ECS task role (Growth) |
| **Purpose** | Storage and runtime injection of LLM provider API keys, DB credentials, OAuth client secrets, and JWT signing keys (`13_System_Architecture.md` §15.3). |
| **Why Selected** | Native AWS integration (no separate service to deploy/secure); IAM-based access control means *only* the API/worker roles can read the secrets they need, enforced by AWS itself rather than application code; rotation support for DB credentials integrates with RDS. |
| **Alternatives Considered** | **HashiCorp Vault (self-hosted)** — operational overhead (a stateful service requiring its own unsealing/backup/HA story) not justified for this team's secret volume (~5–10 secrets); becomes relevant only if multi-cloud or on-prem deployment is required by an enterprise customer in Phase 3. **`.env` files + `python-dotenv`** — acceptable for **local development only** (Section 8) — explicitly **never** for any deployed environment; only a committed `.env.example` template is permitted in version control. |
| **Advantages** | Zero additional infrastructure; integrates with the existing IAM/EC2/ECS model the team already uses for AWS Fundamentals (`01_Project_Framework.md`); audit trail via CloudTrail "for free." |
| **Disadvantages** | Per-secret cost (~$0.40/secret/month) is a (small) recurring line item that grows linearly with secret count — not a concern at this team's scale but worth tracking if the secret count grows unexpectedly (e.g., one API key per LLM provider per environment). |
| **Cost Impact** | ~$2/month at MVP (≈5 secrets). |
| **Learning Curve** | Low — team has AWS Fundamentals; `boto3`'s Secrets Manager client is a few lines. |
| **Future Scalability** | Scales linearly with secret count, no architectural ceiling at this team's scale; if multi-cloud ever becomes necessary, Vault is the documented fallback. |

## 6.4 Vulnerability Scanning — Trivy + Semgrep + pip-audit/npm audit + Dependabot — **EXPANSION (TECH-ADR-006)**

| Attribute | Detail |
|---|---|
| **Selected Technology** | **Trivy** (container image scanning — NEW), **Semgrep** (SAST — NEW), **pip-audit** + **npm audit** + **Dependabot** (dependency scanning, restated from `13_System_Architecture.md` §15.4) |
| **Purpose** | Catch known-CVE dependencies, vulnerable container base images, and common code-level security anti-patterns *in PreFlight's own codebase and images* — "dogfooding" the security posture PreFlight sells to customers. |
| **Why Selected** | `13_System_Architecture.md` §15.4 specified Dependabot + `pip-audit`/`npm audit` but did not address **container image scanning** (particularly important given the simulation sandbox image is explicitly the highest-risk component, §15.2) or **SAST** for application code. **Trivy** scans the `preflight-api`/`preflight-worker`/sandbox Docker images for known OS-package and language-dependency CVEs, is free, fast, and has a GitHub Action emitting **SARIF** — the same output format PreFlight's own product uses for its customers' GitHub Security tab (ADR-008), making "Trivy found an issue in our own sandbox image" a genuinely usable dogfooding narrative. **Semgrep** covers SAST for both the Python and TypeScript codebases with one tool (vs. Bandit, which is Python-only and has a narrower ruleset) — Semgrep's community rule packs cover SQL injection via string formatting, SSRF, and hardcoded secrets, and a SAST finding in PreFlight's own code is a complementary credibility signal alongside PreFlight finding issues in *customer* agent configs (relevant to the Compliance Officer persona, `12_Product_Requirements_Document.md` §4.3). |
| **Alternatives Considered** | **Snyk** — directly analogous to PreFlight's own "Snyk for AI agent permissions" positioning (`12_Product_Requirements_Document.md` §2), and an excellent product — but Snyk's free tier has scan-volume limits a growing monorepo could exceed, introducing a cost line item; Trivy + Semgrep + Dependabot together cover Snyk's MVP-relevant functionality (container, SAST, dependency scanning) at $0. **Bandit** — Python-only SAST; Semgrep's broader language coverage (covers `preflight-web`'s TypeScript too) with one tool and one config format is preferred for a small team that wants to minimize the number of security-tool configurations to maintain. **GitHub Advanced Security (CodeQL)** — excellent, but full private-repo SAST requires GitHub Enterprise in some plan tiers; revisit if/when the team's GitHub plan includes it — would be additive to, not a replacement for, Semgrep. |
| **Advantages** | All four tools are free/open-source at this team's scale; Trivy's SARIF output is the *same mechanism* PreFlight's product (ADR-008) uses, reinforcing the dogfooding narrative; Semgrep's default rule packs require zero custom-rule authoring to provide value on day one. |
| **Disadvantages** | Four separate tools means four configuration surfaces (though all are low-maintenance, mostly-default configs); Semgrep custom rule authoring (if pursued later, e.g., PreFlight-specific anti-patterns like "never log raw LLM prompts containing config secrets") has a moderate learning curve — but is optional, not required for baseline value. |
| **Cost Impact** | $0 — all tools free/open-source at this scale. |
| **Learning Curve** | Low–Medium — Trivy and Dependabot are near-zero-config; Semgrep's default rule packs require no authoring; custom Semgrep rules (optional) are the only Medium-effort item. |
| **Future Scalability** | All four scale with the codebase without licensing-tier concerns at this team's size. If/when GitHub Enterprise is adopted (unlikely pre-Series-A), CodeQL becomes a natural *addition* alongside Semgrep, not a replacement. |

---

# 7. Cloud Stack

## 7.1 AWS Services

| Attribute | Detail |
|---|---|
| **Selected Technology** | EC2 (MVP compute), RDS Postgres, S3, CloudFront, ALB, Secrets Manager, IAM, Route 53, ACM, ECS Fargate (Growth), ElastiCache (Growth), and **AWS SES (NEW — gap-fill, TECH-ADR-008)** for transactional email |
| **Purpose** | Hosting `preflight-api`, `preflight-worker`, `preflight-web`, and supporting data stores, per `13_System_Architecture.md` §17. |
| **Why Selected** | Restated from `13_System_Architecture.md` §17: team's existing AWS Fundamentals skill (`01_Project_Framework.md`); region `ap-south-1` (Mumbai) for India-based-team latency. **SES gap-fill:** neither `12_Product_Requirements_Document.md` nor `13_System_Architecture.md` addresses email delivery — but the custom auth model (Section 6.1) requires it for password-reset flows, and G2 (design-partner onboarding) and scan-completion notifications are natural future uses. SES is the natural choice for the same "fewer vendors" reasoning applied throughout this document — it integrates natively with the existing AWS account/IAM model rather than introducing a third-party (SendGrid/Postmark) account and API key to manage. SES costs approximately $0.10 per 1,000 emails. **Action item:** SES accounts start in a sending-limit "sandbox" mode requiring a limit-increase request that can take 24+ hours to process — the team should submit this in Month 1 to avoid a late surprise when password-reset emails are needed for the first external (design-partner) user. |
| **Alternatives Considered** | **GCP** — team has no GCP experience; Vertex AI's agent tooling is attractive in the abstract, but the team's hosted-LLM choice (Anthropic via LiteLLM, Section 3.3) is cloud-agnostic, so GCP's main AI-specific draw doesn't apply here. **Azure** — similarly, Azure's main draw (native OpenAI integration) is also reachable via LiteLLM from AWS; team's existing skill is AWS. **A single VPS provider (DigitalOcean, Hetzner)** — cheaper raw compute, but lacks the managed-service ecosystem (RDS, Secrets Manager, ECS) that reduces a 3-person team's operational burden; AWS Activate startup-credit programs are also a meaningful consideration that VPS providers' equivalent programs don't match as generously. **SendGrid/Postmark (email)** — both excellent, but a third vendor/API-key to manage for a function SES covers natively; revisit only if SES's deliverability reputation becomes an issue at scale (a Phase 2+ concern). |
| **Advantages** | One cloud account, one IAM model, one billing relationship; managed services (RDS, Secrets Manager, SES) reduce what a 3-person team must operate directly; AWS startup credit programs are a real, non-trivial cost offset for an early-stage team. |
| **Disadvantages** | Vendor lock-in to AWS-specific services (discussed as Section 13, Risk #9) — an accepted tradeoff for a startup at this stage; multi-cloud abstraction would be premature optimization. |
| **Cost Impact** | See Section 12 for the full breakdown — AWS services collectively dominate the non-LLM portion of monthly spend. |
| **Learning Curve** | Low for services the team already knows (EC2, S3, RDS basics per `01_Project_Framework.md`); Medium for ECS Fargate (Growth phase) and Secrets Manager's IAM-policy nuances — both are well-documented and have abundant Terraform module examples (Section 9.3). |
| **Future Scalability** | The MVP→Growth topology transition (`13_System_Architecture.md` §17.1) is the documented scalability path; SES scales linearly with email volume with no architectural change. |

## 7.2 Networking

| Attribute | Detail |
|---|---|
| **Selected Technology** | VPC with public/private subnets, ALB with TLS via ACM, CloudFront for static frontend delivery, security groups restricting DB/Redis to the private subnet (`13_System_Architecture.md` §17.1 diagram) |
| **Purpose** | Network isolation and TLS termination for all PreFlight services. |
| **Why Selected** | Restated from `13_System_Architecture.md` §17 — standard, well-understood AWS networking pattern; ACM provides free, auto-renewing TLS certificates for the ALB; CloudFront caches the static frontend bundle at the edge (near-zero serving cost, per §19.2). |
| **Alternatives Considered** | **API Gateway + Lambda (serverless)** — rejected: the workload (long-running Celery workers, Docker-in-Docker for the simulation sandbox per `13_System_Architecture.md` §9) does not fit Lambda's execution model (15-minute maximum duration, no persistent background processes, no Docker-socket access) — ECS Fargate (already the documented Growth target) is the correct "serverless-ish" destination, not Lambda. |
| **Advantages** | Standard, Terraform-module-supported pattern (Section 9.3); private subnets for DB/Redis/workers mean no direct internet exposure for the most sensitive components. |
| **Disadvantages** | None material beyond the general AWS-networking learning curve, which the team's AWS Fundamentals already partially covers. |
| **Cost Impact** | ALB: ~$16-20/month base + data processing; CloudFront/S3: <$1-2/month at MVP traffic (often within AWS free-tier allowances for the first 12 months). Included in Section 12's totals. |
| **Learning Curve** | Medium — VPC/subnet/security-group design is a one-time setup cost, well-covered by Terraform modules (Section 9.3) and AWS Fundamentals training. |
| **Future Scalability** | The same VPC/ALB topology hosts both the MVP (single EC2 target group) and Growth (ECS Fargate service target group) — no networking redesign between phases. |

## 7.3 Monitoring — CloudWatch + Sentry — **EXPANSION (TECH-ADR-007)**

| Attribute | Detail |
|---|---|
| **Selected Technology** | AWS CloudWatch (infrastructure metrics, restated from `13_System_Architecture.md` §16) + **Sentry (NEW)** for application-level error tracking |
| **Purpose** | CloudWatch: CPU, queue depth, RDS connections, LLM error rates (`13_System_Architecture.md` §16.1's alarm table). Sentry: aggregated exception tracking with stack traces, breadcrumbs, and release tagging across both the FastAPI backend and the React frontend. |
| **Why Selected (Sentry)** | CloudWatch Logs/metrics are excellent for *infrastructure* health but require manually grepping structured logs to find and diagnose application-level *exceptions* — slow for a 3-person team without a dedicated SRE. Sentry automatically groups/deduplicates exceptions, captures stack traces and request context, and — critically for PreFlight's many-async-hop architecture (API → Celery → Docker sandbox → S3 → back to API → frontend poll) — correlates a frontend-visible error with the backend exception that caused it, in one dashboard. `sentry-sdk[fastapi]` and `@sentry/react` cover both halves of the stack with one account. |
| **Alternatives Considered** | **Rollbar / Bugsnag** — similar feature set; Sentry has the largest community and a credible self-hosted option if the free tier is ever outgrown without wanting to pay — lower switching risk. **CloudWatch Logs Insights alone** — workable, but materially slower for day-to-day debugging; this is a "developer velocity" cost that compounds over 15 months rather than a hard blocker — Sentry's free tier removes the tradeoff entirely at MVP scale. **Self-hosted Grafana + Loki/Tempo** — `13_System_Architecture.md` §16 already notes this as a "Phase 2 cost-justified upgrade"; Sentry is complementary (application errors) rather than a replacement (infrastructure metrics/traces), and is zero-ops at MVP scale where Grafana/Loki would not be. |
| **Advantages** | Free tier (5,000 errors/month) is generous relative to MVP traffic; cross-stack (frontend+backend) error correlation; release-tagging makes "did this regression ship in v1.4.2?" a one-click question. |
| **Disadvantages** | A second monitoring dashboard (alongside CloudWatch) — mitigated by Sentry's Slack integration funneling alerts into the same channel as CloudWatch alarms (`13_System_Architecture.md` §16.1's SNS→Slack pattern), so the team has one notification surface even with two underlying tools. |
| **Cost Impact** | $0 at MVP scale (free tier); Sentry Team plan (~$26/month) if error volume grows past the free tier in Growth phase. |
| **Learning Curve** | Low — SDK integration is a few lines per app; the dashboard UI is intuitive. |
| **Future Scalability** | Sentry's paid tiers scale with event volume; if self-hosting ever becomes cost-justified (high event volume + cost-sensitivity), Sentry's open-source self-hosted distribution is a documented migration path with no SDK changes required. |

## 7.4 Logging — structlog + CloudWatch Logs (Restated from `13_System_Architecture.md` §16)

| Attribute | Detail |
|---|---|
| **Selected Technology** | `structlog` (structured JSON logging) + CloudWatch Logs (aggregation, MVP) |
| **Purpose** | Operational logs across `preflight-api`/`preflight-worker`, including LLM call metadata (NFR-9) with secrets/PII redaction. |
| **Why Selected** | Restated from `13_System_Architecture.md` §16 — `structlog` provides structured (JSON) logging with far less boilerplate than manually formatting `logging` records as JSON; CloudWatch Logs requires zero additional infrastructure and integrates with the CloudWatch alarms already specified (§16.1). |
| **Alternatives Considered** | **Standard library `logging` + manual JSON formatter** — `structlog` is effectively "logging done right" for structured output with a much more ergonomic API (context binding, processors) — low switching cost, high ergonomic gain. **Self-hosted ELK (Elasticsearch/Logstash/Kibana)** — rejected on the same operational-overhead grounds as self-hosted Grafana/Loki (Section 7.3); CloudWatch Logs Insights is adequate for MVP log-query needs with zero additional infrastructure. |
| **Advantages** | Structured logs are directly queryable in CloudWatch Logs Insights; `structlog`'s processor pipeline is the natural place to implement NFR-9's secret/PII redaction *before* a log line is emitted, rather than redacting after the fact. |
| **Disadvantages** | None material — this is the "boring, correct" choice. |
| **Cost Impact** | CloudWatch Logs ingestion/storage cost scales with log volume — typically a few dollars/month at MVP scale, included in Section 12's "data transfer/misc" line. |
| **Learning Curve** | Low — `structlog`'s API is a thin, well-documented layer over familiar logging concepts. |
| **Future Scalability** | If log volume/query needs outgrow CloudWatch Logs Insights (Phase 2+), the Grafana/Loki upgrade path (§16) is additive — `structlog`'s JSON output is consumable by either backend without application changes. |

---

# 8. Development Environment — Local Setup

## 8.1 Local Development Stack

| Attribute | Detail |
|---|---|
| **Selected Technology** | Docker Engine + Docker Compose, `uv`, Node.js LTS + npm, host-installed Ollama, `just` (task runner — **NEW, TECH-ADR-014**) |
| **Purpose** | A reproducible local environment for all three engineers across (likely heterogeneous) consumer laptops, matching the production `docker-compose`-based MVP topology as closely as practical. |
| **Why Selected** | **Docker Compose** (dev profile) defines `api`, `worker-static`, `worker-sim`, and `redis` services — the same composition used in production (Section 9.1/11), so "works in dev" closely predicts "works in prod." **Ollama is host-installed, not containerized** — GPU passthrough to Docker containers on Windows/consumer laptops (the team's likely dev OS mix) is non-trivial and not worth the friction; containerized services reach the host's Ollama instance via `host.docker.internal:11434`. **`just`** (a modern `make` alternative — NEW decision, TECH-ADR-014) replaces `make`/Makefiles for `dev`, `test`, `lint`, `migrate` targets: simpler syntax, cross-platform (no tab-vs-space footguns that plague `make` on mixed Windows/macOS/Linux teams), and a single `justfile` documents the entire dev workflow as executable documentation. |
| **Alternatives Considered** | **`make`/Makefile** — the traditional choice, but `make`'s tab-sensitive syntax and inconsistent behavior across `gmake`/`bsdmake`/Windows is a recurring source of "works on my machine" friction for a team likely using mixed OSes; `just` solves this with a near-identical mental model and zero downside for a 3-person team. **Kubernetes/`minikube` for local dev** — explicitly rejected: massive overkill for a 3-person team's MVP; the "operational tax" theme recurs again — `k3s`/`minikube` setup/learning alone would consume meaningful Month 1–2 time better spent on Modules A–C. **Containerized Ollama with GPU passthrough** — rejected for the reason above (host-installed is simpler and avoids a known source of platform-specific Docker-GPU configuration pain). |
| **Advantages** | `docker-compose.yml` dev profile mirrors prod (Section 11), reducing "it worked locally" surprises; `just` provides a single, version-controlled, self-documenting entry point for every common task; host Ollama avoids GPU-in-Docker configuration entirely. |
| **Disadvantages** | Host-installed Ollama means each developer must independently install/update Ollama and pull models — a small, one-time-per-developer setup step (documented in the repo's `CONTRIBUTING.md`/`README.md`). |
| **Cost Impact** | $0. |
| **Learning Curve** | Low — Docker Compose is already a hard dependency (sandbox architecture, Section 9.1); `just`'s syntax is deliberately `make`-like for anyone with prior `make` exposure, and trivial for anyone without. |
| **Future Scalability** | The dev `docker-compose.yml` and the prod `docker-compose.yml` (Section 11) share a common base via Compose's `extends`/override-file mechanism — as services are added (e.g., a new worker queue in Phase 2), both environments update from one source of truth. |

## 8.2 Environment Configuration & Seed Data

| Attribute | Detail |
|---|---|
| **Selected Technology** | `.env.example` (committed) / `.env` (gitignored, local only) via `python-dotenv` for local dev; a seed-data script populating one project with a real open-source MCP server config |
| **Purpose** | Reproducible local configuration without committing secrets (Section 6.3), and immediate frontend/UI development without first standing up a working end-to-end scan pipeline. |
| **Why Selected** | `.env` files are the de facto standard for local-only configuration and are explicitly scoped to **local development only** — Secrets Manager (Section 6.3) is authoritative for every deployed environment. The seed-data script uses one of the real MCP server manifests from the corpus being built per `14_AI_Architecture.md` §21.1 — giving frontend work (Cytoscape graph rendering, findings tables) real, representative data from Week 1, decoupled from the AI pipeline's implementation timeline. |
| **Alternatives Considered** | **Hand-written mock JSON fixtures** — viable as a stopgap, but real MCP server manifests (already a Month 2–3 deliverable per `14_AI_Architecture.md` §21.1) are *more* realistic and serve double duty as parser test fixtures — pulling this forward to Week 1 for one or two manifests is low-cost and high-value for frontend velocity. |
| **Advantages** | Frontend and backend/AI work can proceed in parallel from Week 1 without an artificial "wait for the pipeline" dependency; `.env.example` documents every required configuration variable in one place, easing onboarding. |
| **Disadvantages** | Seed data must be kept in sync with evolving schema (`GraphSnapshot`/`Finding` shapes) as Modules A–D are implemented — a minor maintenance task, mitigated by the seed script being a thin wrapper around the real parser/graph-builder code (so schema changes propagate automatically). |
| **Cost Impact** | $0. |
| **Learning Curve** | Low. |
| **Future Scalability** | N/A — purely a development-time convenience; no production analog. |

---

# 9. DevOps Stack

## 9.1 Docker — docker-compose (Dev/MVP) + Per-Simulation Sandbox Containers

| Attribute | Detail |
|---|---|
| **Selected Technology** | `docker-compose.yml` defining `api`, `worker-static`, `worker-sim`, `redis` services (dev and MVP-prod, Section 11); per-simulation sandbox containers (`13_System_Architecture.md` §9.2) spawned dynamically by `worker-sim` via the Docker SDK for Python, **separate from** the Compose-managed services |
| **Purpose** | Containerization for application services (dev parity, Section 8.1) and for the hardened, ephemeral, no-network sandbox each simulation run executes in. |
| **Why Selected** | Restated/integrated from `13_System_Architecture.md` §9.2 and §17.1/ADR-006 — Docker is already a hard dependency for the sandbox; using Docker Compose for the application services too means the team learns and operates **one** containerization toolchain, not two. |
| **Alternatives Considered** | **Docker Swarm** — rejected: ECS Fargate is the documented Growth-phase orchestration target (`13_System_Architecture.md` §17.1); Swarm would be a third orchestration paradigm (after Compose and Fargate) to learn and then abandon. **Kubernetes (`k3s`) even for the single-EC2 MVP** — rejected for the same operational-overhead reasoning as local `minikube` (Section 8.1) — k8s's value (declarative multi-node orchestration, autoscaling) is irrelevant on a single host and its operational learning curve would consume Month 1–2 time disproportionately. |
| **Advantages** | One containerization toolchain across dev and MVP-prod; the sandbox-container lifecycle (provision → run → teardown, `14_AI_Architecture.md` §9.1) is cleanly separate from the long-lived Compose services, matching the architectural separation already documented. |
| **Disadvantages** | Compose's single-host model means the MVP has no built-in multi-host redundancy — an accepted, documented MVP tradeoff (ADR-006; Section 13, Risk #2's broader single-EC2 discussion). |
| **Cost Impact** | $0 (Docker Engine/Compose are free). |
| **Learning Curve** | Low–Medium — team likely has basic Docker exposure; Compose's YAML is approachable; the Docker SDK for Python (sandbox lifecycle) is a new but well-documented API. |
| **Future Scalability** | Compose services → ECS Fargate task definitions (Section 11/`13_System_Architecture.md` §17.1) is the documented Growth path; sandbox containers migrate to per-simulation Fargate tasks (ADR-003) at the same inflection point. |

## 9.2 CI/CD — GitHub Actions

| Attribute | Detail |
|---|---|
| **Selected Technology** | GitHub Actions, with a per-PR pipeline (lint, unit tests, Classifier Eval Suite, Semgrep, `pip-audit`/`npm audit`) and a nightly pipeline (Known-CVE Detection Suite, Trivy image scans, Playwright E2E suite) |
| **Purpose** | Continuous integration/deployment for `preflight-core`/`api`/`worker`/`cli`/`web`, and the CI gating strategy specified in `14_AI_Architecture.md` §20.1–20.2. |
| **Why Selected** | GitHub OAuth (Section 1.5/6.1) and the GitHub Action CI integration (Module E, `13_System_Architecture.md` §14) already make GitHub the team's hub — GitHub Actions keeps source hosting, OAuth, and CI on one platform with one set of credentials/permissions. GitHub Actions' free tier (2,000 minutes/month for private repos; **unlimited** for public repos) is likely sufficient for MVP needs, and becomes a non-issue entirely if `preflight-core` is open-sourced early per the monetization plan (`11_Final_Startup_Selection.md` §11 step 1) — public-repo Actions minutes are unlimited, a genuinely happy side effect of the open-core strategy. |
| **Alternatives Considered** | **GitLab CI** — would mean either splitting source hosting (GitHub) from CI (GitLab), or migrating off GitHub entirely — losing the OAuth/Action synergy above for no compensating benefit. **CircleCI** — solid product, but its free tier is more restrictive than GitHub Actions' for likely usage patterns, and it's a second platform/credential set. **Jenkins (self-hosted)** — operational overhead (a server to patch, back up, and secure) inconsistent with the "minimize operational surface" philosophy — Jenkins's flexibility advantage doesn't address a need this team actually has. |
| **Advantages** | One platform for source, OAuth, CI, *and* the customer-facing GitHub Action integration (Module E) — maximal synergy; free tier likely covers MVP; SARIF outputs from Trivy/Semgrep (Section 6.4) surface directly in GitHub's Security tab, which is also literally the mechanism PreFlight sells to customers (ADR-008) — the team's own CI becomes a live example of the product's value. |
| **Disadvantages** | GPU-dependent jobs (none currently planned — embeddings and local-model inference are CPU-only per Sections 3.2/3.4) would require self-hosted runners; not a concern given current architecture, but worth noting if a future CI job needs local-model inference at meaningful scale. |
| **Cost Impact** | $0 at MVP (free tier, especially if `preflight-core` is public). |
| **Learning Curve** | Low — GitHub Actions YAML is widely documented; the per-PR/nightly split is a standard pattern. |
| **Future Scalability** | Self-hosted runners (e.g., on the same EC2/ECS infrastructure) are a documented option if Actions minutes ever become a real constraint (Growth phase) — a configuration addition, not a platform migration. |

## 9.3 Infrastructure as Code — Terraform

| Attribute | Detail |
|---|---|
| **Selected Technology** | Terraform (OSS), with S3 backend for remote state |
| **Purpose** | Declarative management of all AWS infrastructure (Section 7) — VPC, EC2/ECS, RDS, S3, CloudFront, ALB, Secrets Manager, Route 53, ACM. |
| **Why Selected** | Restated from `13_System_Architecture.md` §17, with explicit consideration of the Python-native alternative below. The infrastructure described in Section 7 is fairly standard (RDS, ECS, ALB, S3, CloudFront — no exotic conditional-infra-logic requirements that would showcase a general-purpose-language IaC tool's main advantage), and Terraform's module registry has a deep catalog of pre-built, battle-tested modules for exactly these services — meaningfully reducing the amount of infrastructure code the team writes from scratch. |
| **Alternatives Considered** | **AWS CDK (Python)** — genuinely tempting given the team's Python skills (`01_Project_Framework.md`) — CDK lets you define infrastructure in Python rather than HCL. However: (a) CDK ultimately synthesizes to CloudFormation, whose drift-detection and `plan`-equivalent ergonomics have historically lagged Terraform's `terraform plan`/state-file workflow; (b) CDK's main advantage — expressing complex conditional infrastructure logic in a real programming language — isn't needed for this fairly standard infrastructure; (c) learning CDK's *construct* abstractions is its own learning curve, largely offsetting the "it's just Python" appeal. **Decision note (documented fallback):** if the team finds HCL genuinely obstructive after a Month-1 trial, CDK is the fallback — **not** Pulumi. **Pulumi** — a similar Python-native pitch to CDK with Terraform-like multi-cloud breadth, but a smaller community/module ecosystem than both Terraform and CDK — no compelling advantage for this use case over either alternative. |
| **Advantages** | Mature `terraform plan`/state-file workflow; deep module registry for RDS/ECS/ALB/S3/CloudFront reduces hand-written infrastructure code; large community means most error messages have a documented solution. |
| **Disadvantages** | HCL is a new, infrastructure-specific language for a team whose strengths are Python/JS (`01_Project_Framework.md`) — a real, if modest, learning-curve cost; mitigated by the module-registry approach (the team writes mostly module *invocations* with variables, not raw resource definitions). |
| **Cost Impact** | $0 (Terraform OSS; S3 backend for state costs cents/month — already covered by the S3 bucket in Section 5.3). |
| **Learning Curve** | Medium — HCL syntax itself is simple, but understanding Terraform's state model, module composition, and AWS-provider resource schemas takes real time; budget a Month-1 "infrastructure spike" to stand up the MVP topology (Section 11) end-to-end via Terraform before any application code is deployed. |
| **Future Scalability** | The same Terraform codebase manages both the MVP (single-EC2) and Growth (ECS Fargate) topologies (`13_System_Architecture.md` §17.1) via module parameterization/workspaces — the migration is a `terraform apply` with updated module inputs, not a new IaC project. |

---

# 10. Testing Stack

## 10.1 Unit Testing — pytest + pytest-cov + hypothesis

| Attribute | Detail |
|---|---|
| **Selected Technology** | `pytest` (test runner), `pytest-cov` (coverage), `hypothesis` (property-based testing — NEW, TECH-ADR-009) |
| **Purpose** | Unit tests for `preflight-core` (graph construction, path algorithms, risk scoring, canary-token matching, parser logic) and `preflight-api` (FastAPI endpoint tests via `httpx.AsyncClient`). |
| **Why Selected** | `pytest` is the undisputed Python testing standard — fixtures, parametrize, and plugin ecosystem cover every test pattern in this codebase without ceremony. **`hypothesis` is a new, targeted decision** for three specific components where property-based testing materially reduces the hand-written test-case burden: (a) the graph construction pipeline (invariants: "a graph built from any valid config always has at least one Agent node; every Tool node has an outgoing edge to a DataResource or ExternalSink or Credential"), (b) the canary-token matcher (Section 6 of `13_System_Architecture.md` §10 / `14_AI_Architecture.md` §10.4: "no valid canary-token input should cause a matcher exception"), and (c) the risk-scoring formula (invariants: "composite_risk_score is always in [0, 10]; a `SUCCESS` outcome never produces a lower composite score than the static score for the same path"). Property-based testing is the right tool for these invariant-rich, stateful components — and also happens to be an excellent research-and-recruiter signal (FAANG's distributed-systems teams prize it highly). |
| **Alternatives Considered** | **`unittest` (stdlib)** — rejected in favor of pytest for the standard reasons (more ergonomic assertions, fixture DI, plugin ecosystem). **`unittest.mock` (for isolation)** — retained as the *mocking* library (used inside pytest tests via `pytest-mock`) rather than replacing it with a third-party mock framework; `unittest.mock` is the stdlib answer and `pytest-mock`'s thin wrapper adds the `mocker` fixture ergonomics without replacing the mock model. |
| **Advantages** | `hypothesis` catches edge cases that hand-written parametrize tables miss — particularly important for the graph pipeline, where a malformed config file that *technically* parses but produces a degenerate graph is exactly the kind of input a customer might supply and a table-driven test would never have enumerated. `pytest-cov`'s HTML report provides coverage feedback without a separate CI step. |
| **Disadvantages** | `hypothesis` has a learning curve (the `@given` + `st.` strategy model) and initially produces many "shrinking" counterexamples that look mysterious until the model is understood; budget ~1 day of onboarding per engineer. Hypothesis-generated tests are also inherently slower than fixed-input tests because they explore a search space — budget them for the nightly pipeline (Section 9.2), not the per-PR fast suite. |
| **Cost Impact** | $0. |
| **Learning Curve** | Low for pytest; Medium for hypothesis (the property-based mental model is the only genuinely non-trivial concept). |
| **Future Scalability** | `pytest-xdist` (Section 3.5) parallelizes all pytest tests including hypothesis runs across cores; hypothesis's database of discovered-counterexample shrinks accumulates across runs (stored in `.hypothesis/`) — the longer the tests run, the more targeted the search becomes, which is a compounding quality dividend over 15 months. |

## 10.2 Integration Testing — pytest + TestContainers-Python + `httpx.AsyncClient`

| Attribute | Detail |
|---|---|
| **Selected Technology** | `pytest` with `testcontainers-python` for spinning up ephemeral Postgres/Redis instances, and `httpx.AsyncClient` for testing FastAPI routes against a live app instance |
| **Purpose** | Integration tests that cover the boundaries between `preflight-core` (graph/scoring logic) and `preflight-api` (HTTP contract), the DB schema (Alembic migrations applied to a real Postgres container), and Celery task dispatch (Redis container). |
| **Why Selected** | **`testcontainers-python`** starts a real Postgres container (via Docker) for each integration test session and tears it down afterward — testing against real Postgres (with its specific JSONB behavior, index behavior, and transaction semantics) rather than SQLite-in-memory catches a class of migration/schema bugs that in-memory databases systematically miss. This pairs cleanly with the pre-existing Docker Engine dependency (Section 9.1) — testcontainers uses Docker Engine directly. **`httpx.AsyncClient`** with FastAPI's `app` in-process (no actual HTTP port) gives a "real API, real DB, real queue" integration test without a full deployed environment — the documented middle layer between unit tests (mocked everything) and E2E tests (Section 10.4). |
| **Alternatives Considered** | **SQLite-in-memory for integration tests** — explicitly rejected: SQLite lacks JSONB/GIN indexes (the graph-storage mechanism), and Postgres-specific SQL (e.g., the `RETURNING` clause in SQLAlchemy 2.0's async ORM usage, recursive CTEs for Phase 2 graph queries) does not execute in SQLite at all. **`pytest-django` / `pytest-fastapi`** — both are convenience wrappers; direct `httpx.AsyncClient` usage with FastAPI's standard `TestClient`/async equivalent is sufficiently ergonomic without an additional test-framework layer. |
| **Advantages** | Tests against real Postgres/Redis — catches schema bugs, JSONB query bugs, and Alembic migration errors before they reach the deployed MVP; `testcontainers-python` is Docker-backed and cross-platform. |
| **Disadvantages** | Container startup latency (Postgres: ~2–5s, Redis: ~0.5s) means integration tests are slower than unit tests — already accounted for by the per-PR (unit tests only) vs. nightly (all suites) split in Section 9.2. |
| **Cost Impact** | $0. |
| **Learning Curve** | Low–Medium — `testcontainers-python` has a simple API (`PostgresContainer`, `RedisContainer`), and `httpx.AsyncClient` used with pytest fixtures is a well-documented FastAPI pattern. |
| **Future Scalability** | As new integration test scenarios emerge (Phase 2 drift detection, MSSP multi-project queries), the same `testcontainers-python` + `httpx.AsyncClient` pattern extends without architectural changes. |

## 10.3 Security Testing — Bandit + Semgrep (in CI) + Dedicated Simulation-Sandbox Adversarial Tests

| Attribute | Detail |
|---|---|
| **Selected Technology** | **Bandit** (Python AST-based security linter — targeted supplement to Semgrep in CI), **Semgrep** (Section 6.4, already in the CI per-PR pipeline), and a **dedicated adversarial test suite** (`tests/adversarial/`) targeting the Judge's second-order-prompt-injection resilience (`14_AI_Architecture.md` §11.4/§24.1/§24.4) |
| **Purpose** | Static security analysis of PreFlight's own code (Bandit/Semgrep); regression-tests guarding the security controls that *are* PreFlight's differentiating feature (the adversarial suite). |
| **Why Selected** | The adversarial test suite is the most important and most novel entry in this section — per `14_AI_Architecture.md` §24.1, the second-order prompt injection of the Judge (a transcript crafted to manipulate the Judge into reporting `FAILURE` for an attack that actually succeeded) is not just a quality concern but a **P0 safety control for a security product**. A regression in this defense would mean PreFlight tells customers an attack path is safe when it isn't — arguably the worst possible failure mode for a security scanner. These tests run in the nightly suite (they involve real hosted-LLM calls, per Section 9.2's cost gating), with a fixed set of known-malicious transcripts (hand-crafted during development) as regression fixtures. **Bandit** supplements Semgrep for Python-specific patterns Semgrep's community rules may miss (e.g., `subprocess.run(shell=True)` inside the sandbox-lifecycle code, weak random number generation). |
| **Alternatives Considered** | **OWASP ZAP (DAST)** — scanning the running API for HTTP-layer vulnerabilities (SQLi, XSS, authentication misconfigurations) is a valuable *addition* for Phase 2/3 when the SaaS has real external users; explicitly **deferred to Phase 2** (not because it's unimportant but because it requires a deployed instance to scan, and the MVP timeline's priority is the core engine). **Manual red-team exercises** — complement the automated adversarial suite for Phase 2 when design partners are onboarded; scheduled as a ~2-day exercise at the end of Month 9 before the G2 design-partner launch. |
| **Advantages** | The adversarial suite doubles as a *demo* of PreFlight's own security rigor ("we use our own technology's principles to protect our own Judge agent") — a strong narrative for the SIH demo (`11_Final_Startup_Selection.md` §13) and for any security-researcher audit of PreFlight's architecture. |
| **Disadvantages** | Adversarial suite tests involve real LLM calls (the Judge being evaluated) — adding to the nightly LLM API spend (Section 12). The fixed-transcript approach means the suite tests *known* manipulation patterns, not novel ones; new attack-transcript fixtures must be added manually as novel patterns are discovered in practice — a recurring but low-effort maintenance task. |
| **Cost Impact** | ~$1–5/nightly run (Judge-tier LLM calls against the fixed-transcript set, estimated at 20–50 calls per run). |
| **Learning Curve** | Low for Bandit; the adversarial suite requires understanding the Judge's prompt (Section 11 of `14_AI_Architecture.md`) to write meaningful new fixtures — not a separate learning curve so much as a product-knowledge requirement. |
| **Future Scalability** | The adversarial test suite grows with discovered attack patterns; each new pattern added during development, QA, or user-reported feedback becomes a permanent regression fixture — the suite's value compounds over time in exactly the way hypothesis's counterexample database does. |

## 10.4 E2E Testing — Playwright — **NEW DECISION (TECH-ADR-010)**

| Attribute | Detail |
|---|---|
| **Selected Technology** | Playwright (Python or TypeScript driver) for browser-based end-to-end tests of the dashboard flows (login → project creation → config upload → static scan → graph view → finding drill-down) |
| **Purpose** | Catch regressions in the full frontend-backend integration (including the TanStack Query polling, Cytoscape rendering, and shadcn/ui component interactions) that unit and integration tests cannot reach. |
| **Why Selected** | **Playwright** is the current industry standard for browser automation, having largely displaced Selenium in new projects — it supports async Python natively (matching the team's Python-first stack), runs headlessly in CI, and supports all major browser engines (Chromium, Firefox, WebKit) with a single API. The key E2E scenarios for PreFlight's dashboard are well-bounded: login flow, file-upload, graph-renders-with-nodes, findings-list-populates, finding-drill-down-shows-transcript. This is a thin suite (10–20 tests), not comprehensive coverage — the goal is catching "the scan results don't render at all because of a mismatched API response schema" regressions, not recreating the integration test suite in a browser. |
| **Alternatives Considered** | **Selenium** — mature but substantially more boilerplate than Playwright, and its `webdriver`-based architecture introduces flakiness that Playwright's event-based `expect` model largely eliminates. **Cypress** — excellent JavaScript-native DX; rejected because: (a) the team's Python fluency means a Python-driver Playwright suite requires learning only Playwright, not Playwright-equivalent-in-JS; (b) Cypress has historically had limitations with multi-domain requests (less relevant now, but a historical concern for OAuth flows). **No E2E suite at MVP** — a tempting simplification for a 3-person team; ultimately rejected because the SIH demo (`11_Final_Startup_Selection.md` §13) is a live browser demo against a real instance, and an E2E test that confirms "the demo flows work" is directly aligned with a major project goal (A2), not just operational hygiene. |
| **Advantages** | Catches the class of bugs (frontend→backend schema mismatches, API latency-sensitive rendering) that no unit or integration test catches; Playwright's trace-viewer (video + DOM snapshots of each test step) makes CI failures debuggable without reproducing locally. |
| **Disadvantages** | E2E tests are inherently the slowest and most environment-dependent tests — they require a running instance (managed via the CI docker-compose stack, Section 9.1/9.2); flakiness (timeouts, race conditions) is a real ongoing maintenance cost for any E2E suite — mitigated by keeping the suite *thin* (10–20 tests) and using Playwright's `expect`-based auto-waiting rather than `sleep()`. |
| **Cost Impact** | $0 (Playwright is free/open-source). |
| **Learning Curve** | Medium — Playwright's async API and locator model take a few days to become fluent, but its documentation is excellent and the suite will be thin enough that this is a one-time investment. |
| **Future Scalability** | The E2E suite grows with new UC flows (UC-5 compliance export, UC-9 drift detection dashboard) as Phase 2 features ship — a linear, manageable maintenance cost. |

---

# 11. Production Deployment Setup

## 11.1 MVP Topology (Months 1–9 approx. — Single EC2 + docker-compose)

```
Deployment: Single EC2 t3.medium (ap-south-1)
  docker-compose --profile prod up:
    - preflight-api         (FastAPI, Uvicorn, 2 workers)
    - preflight-worker-static  (Celery, prefork pool, concurrency=2)
    - preflight-worker-sim     (Celery, gevent pool, concurrency=4)
    - redis                    (co-located broker + cache)

External (AWS-managed):
    - RDS Postgres db.t4g.micro (private subnet)
    - S3 buckets (transcripts, reports, web-build)
    - CloudFront (CDN for preflight-web static build)
    - ALB (TLS termination via ACM, routes to EC2)
    - Secrets Manager (API keys, DB creds, JWT key)
    - Route 53 (DNS)
    - SES (transactional email, Section 7.1)

Deployment trigger: GitHub Actions `push` to `main` via:
  1. `terraform plan` → `terraform apply` (infra changes, if any)
  2. Build + push Docker images to ECR (preflight-api, preflight-worker)
  3. SSH to EC2: `docker-compose pull && docker-compose --profile prod up -d`
  4. Alembic: `uv run alembic upgrade head` (ran inside API container, post-deploy)
```

**Estimated monthly cost (MVP):** ~$100–140/month. See Section 12 for full breakdown.

**Key operational procedures:**
- **Zero-downtime deploys:** Not achievable on a single EC2 with a single Compose stack (brief downtime during `up -d` for the API container). Acceptable at MVP given the user base; Growth topology (ECS Fargate, Section 11.2) adds rolling updates.
- **Backups:** RDS automated daily backups (7-day retention, included in RDS pricing); S3 versioning on the transcripts bucket; no additional backup setup needed (AWS-managed).
- **Monitoring:** CloudWatch Logs + alarms (Section 7.3/7.4) — alert channels are Slack DMs via SNS→Lambda→Slack webhook, set up in Month 1 before any external users.

## 11.2 Growth Topology (Months 9+ / Phase 2 trigger — ECS Fargate)

```
Deployment: ECS Fargate cluster (ap-south-1)
  Services:
    - preflight-api          (Fargate, task def: 1 vCPU / 2 GB, target tracking autoscaling on ALB request count)
    - preflight-worker-static  (Fargate, task def: 1 vCPU / 2 GB, autoscaling on Celery queue depth via CloudWatch custom metric)
    - preflight-worker-sim     (Fargate Spot, task def: 2 vCPU / 4 GB, autoscaling on simulation queue depth)

External (upgraded):
    - RDS Postgres db.t4g.small+ (or Multi-AZ if SLAs required)
    - ElastiCache Redis cache.t4g.micro (replaces co-located Redis)
    - ECR (Docker image registry, already present at MVP)
    - All other AWS services unchanged from MVP topology

Deployment trigger: GitHub Actions via:
  1. `terraform apply` (ECS task-def updates, if any)
  2. `aws ecs update-service --force-new-deployment` (rolls new image with zero-downtime)
  3. Alembic migration: ECS one-off task (separate from the API service, runs migration before traffic cutover)
```

**Migration trigger from MVP:** determined by the first of these signals, whichever fires first:
- Concurrent scan load makes the single-EC2 CPU/memory boundary visible (>80% utilization sustained > 10 min, per CloudWatch alarm)
- A design partner's SLA requires ≥99.5% uptime (necessitating Multi-AZ at both DB and compute layers)
- Fargate Spot pricing analysis shows a >40% cost saving vs. a vertically-scaled EC2 for the simulation workload (expected if simulation volume grows, given Fargate Spot's typical 60–70% discount vs. on-demand)

---

# 12. Monthly Cost Estimates

All prices are approximate AWS on-demand pricing for `ap-south-1` (Mumbai), unless noted. No reserved instance discounts applied — these are budgeted conservatively at on-demand rates for a startup at this stage; **AWS Activate startup credits** (often $5,000–$25,000 for qualifying early-stage startups) are not assumed but would meaningfully extend the pre-revenue runway.

## 12.1 MVP Phase (Months 1–9 approx.)

| Line Item | Service | Specification | Estimated Cost/Month |
|---|---|---|---|
| Compute | EC2 | `t3.medium` (2 vCPU, 4 GB RAM), on-demand, ap-south-1 | ~$35 |
| Database | RDS Postgres | `db.t4g.micro`, on-demand, 20 GB gp3 storage | ~$15 |
| Cache/Broker | Redis | Co-located on EC2 | $0 incremental |
| Object Storage | S3 | ~50 GB transcripts/reports/static (first-year free tier applies partially) | ~$1–2 |
| CDN | CloudFront | <10 GB/month transfer at MVP scale | ~$1 |
| Load Balancer | ALB | Base + data processing (minimal at MVP) | ~$18 |
| Secrets | Secrets Manager | ~5–8 secrets | ~$2–3 |
| Email | SES | <1,000 transactional emails/month | <$1 |
| LLM API | Anthropic / OpenAI | Development-phase ($50–100/month per `14_AI_Architecture.md` §14.3); first external scan adds $0.10–2.00/scan | $50–100 |
| Container Registry | ECR | <1 GB/month images | ~$0.10 |
| DNS | Route 53 | Hosted zone ($0.50/month) + queries | ~$1 |
| TLS | ACM | Free with ALB | $0 |
| Monitoring/Error | CloudWatch + Sentry | CloudWatch standard metrics/logs; Sentry free tier | ~$5–10 |
| Data Transfer | AWS | Outbound → internet at MVP scale | ~$5 |
| **Total (excluding LLM API)** | | | **~$85–90/month** |
| **Total (with LLM API — development phase)** | | | **~$140–190/month** |
| **Total (with LLM API — design-partner phase, G2)** | | | **~$90–140/month** (lower LLM spend once pipeline dev matures) |

**Cost discipline note:** The single largest controllable cost lever is LLM API spend — the $50–100/development-month estimate reflects active simulation-pipeline development (dozens of test runs/day). Once development matures (~Month 6–7), this drops toward the $10–30/month level for CI nightly runs + design-partner onboarding scans. The per-scan budget cap (NFR-3 / `13_System_Architecture.md` §19.1) means design-partner usage is bounded, not unbounded.

## 12.2 Growth Phase (Months 9+ / Phase 2)

| Line Item | Service | Specification | Estimated Cost/Month |
|---|---|---|---|
| Compute (API) | ECS Fargate | 2× `1 vCPU / 2 GB` tasks, ~730 hrs/month | ~$50 |
| Compute (Workers) | ECS Fargate Spot | Simulation workers, spot pricing (~60–70% discount) | ~$20–40 |
| Database | RDS Postgres | `db.t4g.small` + read replica | ~$40–60 |
| Cache/Broker | ElastiCache | `cache.t4g.micro` | ~$15–20 |
| Object Storage | S3 (+ IA transition) | 200+ GB as scan volume grows, lifecycle policies active | ~$5–10 |
| All other items | (unchanged or marginal growth) | ALB, Secrets, SES, CloudFront, Route 53 | ~$25 |
| LLM API | Anthropic / OpenAI | $0.10–2.00/scan × design-partner + CI volume | $30–100+ (scales with usage) |
| Monitoring | CloudWatch + Sentry | Sentry Team plan if free tier exceeded | ~$30–40 |
| **Total (Growth, est.)** | | | **~$215–325/month** |

**Revenue sustainability note:** at a typical early-stage SaaS pricing of $200–500/month per paying customer (Snyk's pricing model, which PreFlight is explicitly patterned after per `12_Product_Requirements_Document.md` §2), **2–3 paying customers cover the entire Growth-phase infrastructure cost** — a realistic initial sales target to achieve full infrastructure cost coverage before needing external funding for ops, as noted in the startup-potential assessment (`11_Final_Startup_Selection.md` §11).

---

---

# 13. Technology Risks

The following risk register covers the ten highest-priority technology risks specific to this stack. Each is rated by Likelihood × Impact and paired with a concrete mitigation already baked into a prior decision — this is not a standard "list concerns and hope" register, but an accountability checklist for Months 1–3 spikes and ongoing monitoring.

| # | Risk | Likelihood | Impact | Mitigation Already Decided | Owner/Timeline |
|---|---|---|---|---|---|
| **1** | **LLM provider refusal of Attacker-role prompts** (`14_AI_Architecture.md` §6.7, §24.4) — Anthropic's safety filters refuse requests framed as "craft a prompt injection attack" at a high enough rate to break the simulation pipeline | High (unknown until tested) | Critical (T3/G1 blocked) | Month-1 spike: empirically test refusal rates against `claude-sonnet-4-6` for representative Lethal Trifecta scenarios before committing to the Attacker prompt design; if refusal rate > 20%, iterate on system-prompt framing before Month 2 work begins | All three engineers, Month 1 |
| **2** | **Single-EC2 Redis SPOF** — co-located Redis crash simultaneously takes down job dispatch (Celery broker), the classifier cache, and rate limiting; no hot standby at MVP | High (in any non-trivial uptime requirement) | High (complete service outage until Redis restarts) | Accepted MVP tradeoff, documented (ADR-006 / Section 11.1). Mitigation: Docker restart policy (`restart: always`) gives Redis auto-recovery in <30s for crash/OOM; Growth-phase trigger is a design-partner SLA requirement (Section 11.2) | Accepted. Monitor CloudWatch Redis OOM alarm (Section 7.3); Growth trigger is explicit |
| **3** | **Celery + gevent monkey-patching incompatibility** (`14_AI_Architecture.md` §14.3 / Section 2.3) — the simulation worker's I/O concurrency model (`gevent`) conflicts with the Docker SDK for Python or the httpx client used by LiteLLM, causing mysterious hangs or incorrect behavior | Medium | High (blocks T3/G1 simulation pipeline) | Month 2–3 pilot: stand up `worker-sim` with gevent pool, confirm Docker SDK sandbox lifecycle (`docker run`, `docker kill`) and LiteLLM `asyncio`-based calls are both gevent-compatible; documented `prefork` fallback (higher memory, lower concurrency) if gevent proves unstable | All three engineers, Month 2–3 |
| **4** | **LiteLLM version drift breaking tool-call-format normalization** (Section 3.1 / `14_AI_Architecture.md` §7.7) — a LiteLLM release changes its normalization of Anthropic's or OpenAI's tool-call schemas, silently breaking the Victim Agent's tool-calling loop | Medium | High (silent correctness regression — simulation produces wrong transcripts) | Small per-provider conformance test suite (`one fixture per supported provider, per `14_AI_Architecture.md` §7.7), run in the per-PR CI pipeline; pin LiteLLM version in `uv.lock`; treat LiteLLM major-version bumps as requiring explicit conformance-test re-validation | Per-PR CI gate (Section 9.2); version pins in `uv.lock` (Section 2.5) |
| **5** | **MCP specification evolution breaking parsers** (`12_Product_Requirements_Document.md` §13 / `14_AI_Architecture.md` §21.1 update cadence note) — the MCP spec's tool-level RBAC additions (already introduced in the 2026 update per `11_Final_Startup_Selection.md` §5.2) continue evolving, rendering parser assumptions stale | Medium | High (scanner produces incorrect/incomplete permission graphs for new configs) | Versioned schema support (FR-A1.4); the `ConfigParser` plugin interface (`13_System_Architecture.md` §11.1) is designed for new format versions to be added as new modules without touching the graph builder; monthly review of MCP spec changelog (lightweight, already paired with the CVE corpus refresh cron per `14_AI_Architecture.md` §16.5) | One engineer designated "MCP spec monitor" rotating monthly; parser versioning baked into Section 2.4's Pydantic schemas |
| **6** | **Simulation sandbox container escape or resource exhaustion** (`13_System_Architecture.md` §9.2 / §15.2 / `14_AI_Architecture.md` §24) — an LLM-generated tool call or mock-tool response causes the sandbox container to consume excessive CPU/memory, affecting co-located services on the single EC2 host | Low–Medium | High (affects all hosted services; credibility catastrophe for a security product that "escaped" its own sandbox) | Docker sandbox hardening checklist (Section 9.2: `--read-only`, non-root user, `--security-opt=no-new-privileges`, seccomp default profile, `--network=none`, `--cap-drop=ALL`, per-container CPU/memory caps enforced via `--cpus`/`--memory`); wall-clock timeout + `docker kill` fallback; sandbox security is explicitly the `13_System_Architecture.md` §15.2's "highest-risk component" — all changes to `core/simulation/sandbox.py` require security review as a process control | Security review process gate on `core/simulation/sandbox.py` changes from Month 1 |
| **7** | **`uv` workspace immaturity causing dependency-resolution failures** (Section 2.5 / TECH-ADR-001) — a package in the monorepo has an unusual build backend (e.g., `scikit-learn`, native-extension packages like `numpy`) that `uv`'s current resolver handles incorrectly | Low–Medium | Medium (dev environment setup fails for one or more engineers; CI pipeline broken) | Month-1 spike: `uv sync` against the full dependency set including `sentence-transformers`, `networkx`, `torch` (if any path imports it — unlikely given Ollama handles inference), `testcontainers`, before any application code is written; documented fallback to Poetry (Section 2.5) at the cost of ~0.5-1 day migration, feasible only if discovered early | Month 1, before any application code; all engineers |
| **8** | **Anthropic / OpenAI API pricing changes degrading NFR-3 compliance** — a pricing increase for `claude-sonnet-4-6`-class models shifts the per-scan cost above the $1–2 target (NFR-3) without code changes | Low (pricing is generally stable over 6–12 month horizons, and has trended *down* not up for frontier models through 2024–2026) | Medium (either cost-cap enforcer fires more aggressively, reducing simulation depth, or per-scan cost to customers must rise) | Model-routing config (`13_System_Architecture.md` §10.2) is a YAML file — switching attacker/judge to a cheaper equivalent is a config change, not a code change; the cost-cap enforcement in the LLMGateway (Section 3.1 / `14_AI_Architecture.md` §12.2) ensures cost overruns fail *safely* (partial results returned) rather than silently accumulating | Monitor monthly billing; model-routing YAML is the mitigation lever |
| **9** | **AWS vendor lock-in limiting optionality** (Section 7.1) — if AWS Activate credits run out faster than expected or pricing changes significantly, the team's deep AWS-specific choices (RDS, ECS, Secrets Manager) create switching friction | Low (at MVP scale, AWS alternatives are genuinely comparable) | Low–Medium at MVP scale; Higher if the team has grown revenue-dependent before noticing | Accepted as a deliberate startup-stage tradeoff (discussed in Section 7.1). The mitigation is sequenced: Terraform (Section 9.3) manages *all* infrastructure declaratively — a cloud migration is still painful but "IaC-facilitated-painful" rather than "hand-rolled-everything-painful." The application code itself has no AWS-SDK calls outside of `boto3` for S3/Secrets Manager, both of which have clear equivalents on GCP (GCS/Secret Manager) and Azure (Blob Storage/Key Vault) | No active mitigation; accepted tradeoff. Revisit at Series A if GCP/Azure cloud credits or customer requirements create a compelling reason |
| **10** | **Pydantic v2 ecosystem churn causing third-party library incompatibilities** (Section 2.4) — a library PreFlight depends on (e.g., a future Celery extension, a `sentence-transformers` update) drops Pydantic v2 compatibility or pins Pydantic v1 | Low (Pydantic v2 is now well-established and most libraries have migrated) | Medium (dependency resolution failure, requiring a pin workaround) | `uv.lock` pins the full transitive dependency graph — incompatibilities manifest at `uv sync` time (discoverable immediately) rather than at runtime (discoverable only when a specific code path is exercised); GitHub Dependabot (Section 6.4) alerts on updates that would break the lock file | Dependabot alerts + `uv lock --check` in CI |

---

---

# 14. Architecture Decision Records — New Decisions (TECH-ADRs)

The following ADRs cover decisions **newly made in this document** that were not already resolved in `13_System_Architecture.md` (ADR-001–ADR-009) or `14_AI_Architecture.md` (AI-ADR-001–AI-ADR-008). They are numbered starting at TECH-ADR-001 to distinguish them.

| TECH-ADR | Decision | Alternative(s) Considered | Rationale | Section |
|---|---|---|---|---|
| **001** | `uv` (Python) for monorepo dependency management | Poetry (documented fallback), pip+pip-tools | 10–100× install speed; native workspace support for the `core`/`api`/`worker`/`cli` layout; `pip`-compatible CLI; Month-1 spike to confirm compatibility | 2.5 |
| **002** | npm (Node) for the single `preflight-web` package | pnpm, Yarn Berry | One-package repo: `pnpm`'s deduplication advantage doesn't materialize; npm is zero additional tooling; tracked revisit trigger if a second JS package emerges | 2.5 |
| **003** | PyJWT (replacing `python-jose` from `13_System_Architecture.md` §6) | python-jose, Authlib JWT | `python-jose` has a history of algorithm-confusion CVEs and reduced maintenance velocity; PyJWT is more actively maintained with a simpler, less-footgun-prone algorithm specification API — a credibility-critical correction for a security product | 6.1 |
| **004** | `argon2-cffi` used directly (replacing "Argon2 via `passlib`" from `13_System_Architecture.md` §6) | passlib (unmaintained since 2020, breaks with bcrypt ≥ 4.0), bcrypt directly | `passlib` is unmaintained and has a documented compatibility break; since Argon2 is already the chosen algorithm, using `argon2-cffi` directly removes the unmaintained abstraction layer from the most security-critical code path in the application | 6.1 |
| **005** | Lean custom auth implementation (replacing "`fastapi-users` or custom JWT" from `13_System_Architecture.md` §6) | fastapi-users | PreFlight's custom `organizations`/`memberships`/role schema (ADR-006) makes bending `fastapi-users`'s standard user model more work than the ~300–500 lines of custom auth code using PyJWT + Authlib + `argon2-cffi` | 6.1 |
| **006** | Trivy + Semgrep for container image scanning and SAST (supplementing Dependabot + pip-audit/npm audit from `13_System_Architecture.md` §15.4) | Snyk (free-tier limits), GitHub CodeQL (tier-dependent), Bandit-only for Python | Container image scanning is essential for the sandbox image (the highest-risk component per §15.2); Semgrep covers both Python and TypeScript with one tool and one config; SARIF output from Trivy integrates with GitHub Security tab (ADR-008 synergy — dogfoods PreFlight's own CI-integration value prop) | 6.4 |
| **007** | Sentry (application error tracking, supplementing CloudWatch from `13_System_Architecture.md` §16) | CloudWatch Logs Insights only, Rollbar, Bugsnag | Cross-stack (frontend+backend) exception correlation and aggregation materially accelerates debugging for a 3-person team without a dedicated SRE; Sentry's free tier covers MVP scale; CloudWatch remains authoritative for infrastructure metrics; Sentry covers application-level exceptions | 7.3 |
| **008** | AWS SES for transactional email (gap-fill — not addressed in prior documents) | SendGrid, Postmark, Mailgun | SES integrates natively with existing AWS account/IAM model; no third-party vendor account; requires sending-limit sandbox lift (Month-1 action item) | 7.1 |
| **009** | `hypothesis` (property-based testing) for graph, canary, and risk-scoring invariants | Extended pytest parametrize tables, manual edge-case enumeration | Property-based testing is the right tool for invariant-rich, stateful components (graph construction, canary matching, risk formula); catches edge cases hand-written tables miss; compounding quality dividend as the hypothesis counterexample database grows; strong recruiter signal | 10.1 |
| **010** | Playwright for E2E testing | Selenium, Cypress, no E2E suite at MVP | Current industry standard browser automation; async Python driver matches team's stack; SIH demo (A2) alignment justifies a thin E2E suite; Playwright's trace-viewer makes CI failure debugging practical | 10.4 |
| **011** | `just` (task runner) for dev workflow commands | make/Makefile, shell scripts, no task runner | Cross-platform (no tab-vs-space footguns on mixed dev OS environments); near-identical mental model to `make`; single `justfile` is version-controlled, executable documentation | 8.1 |
| **012** | Custom pytest harness for AI evaluation suites; promptfoo as informal dev tool only | DeepEval, Ragas, promptfoo as CI dependency | The three evaluation suites (`14_AI_Architecture.md` §20) are domain-specific (graph+simulation pipeline, not prompt→response pairs); a thin `pytest` parametrize harness integrates with the existing CI split without a second test-runner; promptfoo retained as an informal prompt-iteration tool, not a CI gate | 3.5 |
| **013** | Staged graph-database strategy: NetworkX now → Postgres `WITH RECURSIVE` before any graph DB → Neo4j/Neptune only if Stage 2 measurably fails | Jump directly to Neo4j at ADR-001's revisit trigger | Stage 2 (recursive CTEs) is materially cheaper (zero new infrastructure) and likely sufficient at the scale this team's customer base will realistically reach within 24 months; each stage is adopted only when the previous stage's empirical limits are reached | 4.3 |
| **014** | `testcontainers-python` for real Postgres/Redis in integration tests | SQLite-in-memory, mocked DB layer | SQLite lacks JSONB/GIN (load-bearing for graph storage); Postgres-specific SQL doesn't execute in SQLite; real containers catch migration/schema bugs before deployment; Docker Engine is already a hard dependency (sandbox, Section 9.1) | 10.2 |

---

# 15. Final Technology Decisions — Consolidated Reference

The following table is the single-page canonical answer to "what do we use for X" for all 15 months. All decisions are locked unless a TECH-ADR override process (document a new TECH-ADR, update this table, get all-team sign-off) is followed.

## 15.1 Full Stack Reference

| Category | Layer | Selected Technology | Locked Since |
|---|---|---|---|
| **Frontend** | Framework | React 18 + TypeScript + Vite | `13_System_Architecture.md` §3 |
| | UI Library | Tailwind CSS + shadcn/ui | `13_System_Architecture.md` §3 |
| | State — Server | TanStack Query | `13_System_Architecture.md` §3 |
| | State — Client | Zustand | `13_System_Architecture.md` §3 |
| | Graph Visualization | Cytoscape.js + cytoscape-dagre | `13_System_Architecture.md` §3 |
| | Auth (frontend) | In-memory JWT + httpOnly refresh cookie | `13_System_Architecture.md` §3 |
| **Backend** | API Framework | FastAPI (Python 3.12) + Uvicorn/Gunicorn | `13_System_Architecture.md` §4 |
| | Task Queue | Celery | `13_System_Architecture.md` §4 |
| | Queue Broker | Redis | `13_System_Architecture.md` §4 |
| | Worker — Static | Celery prefork pool | `13_System_Architecture.md` §4 |
| | Worker — Sim | Celery gevent pool | `13_System_Architecture.md` §4 |
| | Validation | Pydantic v2 | `13_System_Architecture.md` §4 |
| | Dependency Mgmt (Python) | `uv` + workspace | **TECH-ADR-001** (this doc) |
| | Dependency Mgmt (JS) | npm | **TECH-ADR-002** (this doc) |
| | Report Generation | WeasyPrint (HTML→PDF) | `13_System_Architecture.md` ADR-007 |
| **AI/ML** | LLM Framework | LiteLLM + custom `LLMGateway` | `13_System_Architecture.md` ADR-002 / `14_AI_Architecture.md` |
| | Local Models | Ollama (Qwen2.5-3B-Instruct Q4, Qwen2.5-1.5B-Instruct Q4) | `14_AI_Architecture.md` AI-ADR-002 |
| | Hosted — Primary | Anthropic (`claude-sonnet-4-6`) | `14_AI_Architecture.md` §13 |
| | Hosted — Classifier Esc. | Anthropic (`claude-haiku-4-5`) | `14_AI_Architecture.md` §13 |
| | Hosted — Secondary | OpenAI (via LiteLLM, Victim + fallback) | `14_AI_Architecture.md` §13 |
| | Embedding Model | sentence-transformers/all-MiniLM-L6-v2 | `14_AI_Architecture.md` AI-ADR-005 |
| | Vector Index | FAISS (in-process, bundled) | `14_AI_Architecture.md` §17 |
| | Prompt Templates | Jinja2 (versioned `.jinja` files) | `14_AI_Architecture.md` §11 |
| | AI Evaluation | Custom pytest harness | **TECH-ADR-012** (this doc) |
| | Prompt Dev Tool | promptfoo (informal, not CI) | **TECH-ADR-012** (this doc) |
| | Phase 2 Fine-tune | QLoRA on cloud GPU (deferred) | `14_AI_Architecture.md` AI-ADR-003 |
| | Phase 2 Risk Model | XGBoost/LightGBM (tabular, deferred) | `14_AI_Architecture.md` §19 |
| **Graph** | Graph Library | NetworkX (`DiGraph`) | `13_System_Architecture.md` ADR-001 |
| | Algorithms | Yen's k-shortest-paths + BFS/DFS (NetworkX), custom PatternDetectors | `14_AI_Architecture.md` §5 |
| | Graph Storage | Postgres JSONB (`graph_snapshots`) | `13_System_Architecture.md` ADR-001/ADR-005 |
| | Future Graph DB | Postgres `WITH RECURSIVE` → Neo4j/Neptune (staged) | **TECH-ADR-013** (this doc) |
| **Database** | RDBMS | PostgreSQL 15+ (RDS) + SQLAlchemy 2.0 + Alembic | `13_System_Architecture.md` ADR-005 |
| | Cache | Redis (co-located MVP → ElastiCache Growth) | `13_System_Architecture.md` §6 |
| | Object Storage | Amazon S3 | `13_System_Architecture.md` §7.3 |
| **Security** | JWT | PyJWT | **TECH-ADR-003** (this doc — corrects `python-jose`) |
| | Password Hashing | argon2-cffi (direct) | **TECH-ADR-004** (this doc — corrects `passlib`) |
| | OAuth | Authlib (GitHub OAuth) | `13_System_Architecture.md` §6.2 |
| | Auth Framework | Custom (~300–500 lines, FastAPI deps) | **TECH-ADR-005** (this doc — corrects `fastapi-users`) |
| | Authorization | Application-layer RBAC (FastAPI deps) | `13_System_Architecture.md` ADR-009 |
| | Secrets | AWS Secrets Manager | `13_System_Architecture.md` §15.3 |
| | Container SAST/CVE | Trivy (SARIF) | **TECH-ADR-006** (this doc) |
| | Code SAST | Semgrep + Bandit | **TECH-ADR-006** (this doc) |
| | Dependency Scanning | Dependabot + pip-audit + npm audit | `13_System_Architecture.md` §15.4 |
| | Rate Limiting | slowapi (Redis-backed) | `13_System_Architecture.md` §5 |
| **Cloud** | Provider | AWS (`ap-south-1`) | `13_System_Architecture.md` §17 |
| | MVP Compute | EC2 `t3.medium` + docker-compose | `13_System_Architecture.md` ADR-006 |
| | Growth Compute | ECS Fargate (incl. Fargate Spot for sim workers) | `13_System_Architecture.md` §17.1 |
| | Networking | VPC + ALB (TLS/ACM) + CloudFront | `13_System_Architecture.md` §17 |
| | Email | AWS SES | **TECH-ADR-008** (this doc) |
| | Monitoring | CloudWatch + Sentry | **TECH-ADR-007** (this doc) |
| | Logging | structlog + CloudWatch Logs | `13_System_Architecture.md` §16 |
| | Error Tracking | Sentry (frontend + backend) | **TECH-ADR-007** (this doc) |
| **DevOps** | Containerization | Docker Engine + Docker Compose + Docker SDK (Python) | `13_System_Architecture.md` §9.2/§17 |
| | CI/CD | GitHub Actions | This doc §9.2 |
| | IaC | Terraform OSS + S3 backend | This doc §9.3 |
| | Task Runner | `just` | **TECH-ADR-011** (this doc) |
| | Container Registry | Amazon ECR | `13_System_Architecture.md` §17 |
| **Testing** | Unit | pytest + pytest-cov + hypothesis | **TECH-ADR-009** (this doc) |
| | Mocking | pytest-mock (wraps unittest.mock) | This doc §10.1 |
| | Integration | pytest + testcontainers-python + httpx.AsyncClient | **TECH-ADR-014** (this doc) |
| | Security | Bandit + Semgrep + adversarial-Judge suite | This doc §10.3 |
| | E2E | Playwright | **TECH-ADR-010** (this doc) |
| | Parallelism | pytest-xdist | Section 3.5 / Section 10.1 |
| **Dev Env** | Python runtime | Python 3.12 (pinned in `.python-version`) | This doc §8 |
| | Node runtime | Node.js LTS (pinned in `.nvmrc`) | This doc §8 |
| | Local LLM serving | Ollama (host-installed) | `14_AI_Architecture.md` AI-ADR-002 |
| | Local dev orchestration | docker-compose (dev profile) | This doc §8.1/§9.1 |
| | Env config | `.env.example` / `.env` (gitignored) via python-dotenv | This doc §8.2 |
| | Seed data | One-time seed script (real MCP server manifest) | This doc §8.2 |

## 15.2 Month-1 Mandatory Spikes (Resolving Open Risks Before Code Is Written)

These are **blocking spikes** — work that must be completed and signed off before the corresponding component enters active development, per the risk register (Section 13) and AI-ADR-008.

| Spike | Blocking Risk | Owner | Deadline |
|---|---|---|---|
| 1. Run `uv sync` against the full dependency graph (including `sentence-transformers`, `networkx`, `scipy`, `testcontainers`) | TECH-ADR-001 maturity / Section 13 Risk #7 | All three engineers | Month 1, Week 1 |
| 2. Empirically test Attacker-role prompt refusal rates: 10+ representative Lethal Trifecta scenarios against `claude-sonnet-4-6` | `14_AI_Architecture.md` §6.7, §24.4 / Section 13 Risk #1 | Designated AI engineer | Month 1, Week 2 |
| 3. Stand up MVP AWS topology via Terraform (EC2 + RDS + S3 + ALB), with `preflight-api` returning a `/health` response behind a real TLS cert | `13_System_Architecture.md` §17 / Section 13 (general) | Designated DevOps engineer | Month 1, Week 3 |
| 4. Submit SES sending-limit increase request | TECH-ADR-008 / Section 7.1 | Any engineer | Month 1, Week 1 |
| 5. Confirm Celery + gevent + Docker SDK compatibility in a minimal sandbox-lifecycle test (`docker run hello-world` from a gevent-pooled Celery task) | Section 13 Risk #3 | Designated backend engineer | Month 2, Week 1 |
| 6. Verify RAG corpus licenses (AgentDojo, InjecAgent, ASB, Agent-SafetyBench) for redistribution in open-core `preflight-core` | `14_AI_Architecture.md` AI-ADR-007 / §21.3 | All three engineers (legal review or community license check) | Month 1–2 |

---

---

# 16. Dependency Manifest — Key Libraries

The following is the canonical list of direct (non-transitive) dependencies across each package in the monorepo, for use during `uv`/npm workspace initialization.

## 16.1 `preflight-core` (Python)

```toml
# pyproject.toml [project.dependencies]
networkx[default] = ">=3.3"          # Graph library (Section 4.1)
pydantic = ">=2.7"                    # Schema validation (Section 2.4)
sentence-transformers = ">=3.0"       # Embedding model (Section 3.4)
faiss-cpu = ">=1.8"                   # Vector index (Section 4.3 / `14_AI_Architecture.md` §17)
litellm = ">=1.40"                    # LLM provider abstraction (Section 3.1)
jinja2 = ">=3.1"                      # Prompt templates (Section 3.1 / `14_AI_Architecture.md` §11)
tenacity = ">=9.0"                    # LLM retry/backoff (Section 3.1)
structlog = ">=24.1"                  # Structured logging (Section 7.4)
python-dotenv = ">=1.0"               # .env loading for local dev only (Section 8.2)
weasyprint = ">=62.0"                 # HTML→PDF report generation (ADR-007)
pyyaml = ">=6.0"                      # scoring_config.yaml / standards_map.yaml / model_routing.yaml
scipy = ">=1.13"                      # Used by sentence-transformers for embedding similarity
```

## 16.2 `preflight-api` (Python)

```toml
# pyproject.toml [project.dependencies]
preflight-core = {workspace = true}   # Path dependency (Section 2.5 / TECH-ADR-001)
fastapi = ">=0.111"                   # API framework (Section 2.1)
uvicorn = {extras = ["standard"], version = ">=0.30"}
sqlalchemy = {extras = ["asyncio"], version = ">=2.0"}
alembic = ">=1.13"                    # DB migrations (Section 5.1)
asyncpg = ">=0.29"                    # Async Postgres driver (Section 5.1)
redis = ">=5.0"                       # Redis client (Section 5.2)
pyjwt = {extras = ["crypto"], version = ">=2.8"}  # JWT auth (TECH-ADR-003)
argon2-cffi = ">=23.1"                # Password hashing (TECH-ADR-004)
authlib = ">=1.3"                     # GitHub OAuth (Section 6.1)
httpx = ">=0.27"                      # Async HTTP client for OAuth callbacks + tests (Section 10.2)
slowapi = ">=0.1.9"                   # Rate limiting (Section 2.1 / `13_System_Architecture.md` §5)
boto3 = ">=1.34"                      # AWS SDK: S3, Secrets Manager, SES (Section 5.3 / 6.3 / 7.1)
celery = {extras = ["gevent"], version = ">=5.4"}  # Task queue (Section 2.2/2.3)
sentry-sdk = {extras = ["fastapi"], version = ">=2.0"}  # Error tracking (TECH-ADR-007)
```

## 16.3 `preflight-worker` (Python)

```toml
# pyproject.toml [project.dependencies]
preflight-core = {workspace = true}
celery = {extras = ["gevent"], version = ">=5.4"}
docker = ">=7.1"                      # Docker SDK for sandbox lifecycle (Section 9.1)
redis = ">=5.0"
boto3 = ">=1.34"                      # S3 transcript writes (Section 5.3)
sentry-sdk = ">=2.0"
structlog = ">=24.1"
```

## 16.4 `preflight-cli` (Python)

```toml
# pyproject.toml [project.dependencies]
preflight-core = {workspace = true}
typer = {extras = ["all"], version = ">=0.12"}  # CLI framework
rich = ">=13.7"                       # Terminal output (Section `13_System_Architecture.md` §14.3)
httpx = ">=0.27"                      # API calls to preflight-api (for simulate command)
pyyaml = ">=6.0"
```

## 16.5 `preflight-web` (JavaScript/npm)

```json
{
  "dependencies": {
    "react": "^18.3",
    "react-dom": "^18.3",
    "@tanstack/react-query": "^5.40",
    "zustand": "^4.5",
    "cytoscape": "^3.29",
    "cytoscape-dagre": "^2.5",
    "cytoscape-popper": "^2.0",
    "react-router-dom": "^6.23",
    "@radix-ui/react-dialog": "^1.1",
    "@radix-ui/react-dropdown-menu": "^2.1",
    "@radix-ui/react-tooltip": "^1.1",
    "tailwindcss": "^3.4",
    "class-variance-authority": "^0.7",
    "clsx": "^2.1",
    "lucide-react": "^0.383"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.3",
    "vite": "^5.3",
    "typescript": "^5.4",
    "playwright": "^1.44",
    "@sentry/react": "^8.0",
    "openapi-typescript": "^7.0",
    "prettier": "^3.3",
    "prettier-plugin-tailwindcss": "^0.6",
    "eslint": "^9.0"
  }
}
```

## 16.6 Test-Only Dependencies (dev extras in each package's `pyproject.toml`)

```toml
[project.optional-dependencies.dev]
pytest = ">=8.2"
pytest-asyncio = ">=0.23"
pytest-cov = ">=5.0"
pytest-xdist = ">=3.5"
pytest-mock = ">=3.14"
hypothesis = ">=6.103"
testcontainers = {extras = ["postgres", "redis"], version = ">=4.7"}  # TECH-ADR-014
bandit = {extras = ["toml"], version = ">=1.7"}
semgrep = ">=1.75"
pip-audit = ">=2.7"
```

---

# 17. Document Conclusion

This document is the **final, official technology reference** for PreFlight's 15-month build. It builds directly on the architectural philosophy (fewer systems, boring tech, 3-person team discipline) established across `11_Final_Startup_Selection.md` through `14_AI_Architecture.md`, and closes all remaining technology gaps while making two corrections to prior documents that a security-product credibility review would otherwise surface (`python-jose` → PyJWT; `passlib` → `argon2-cffi`).

**New decisions introduced (TECH-ADR-001–014):**
- `uv` workspace for Python monorepo dependency management
- `npm` for the single frontend package
- PyJWT replacing `python-jose` (JWT security correction)
- `argon2-cffi` direct replacing `passlib` (unmaintained library correction)
- Custom auth replacing `fastapi-users` (custom schema fit)
- Trivy + Semgrep for container image scanning and SAST
- Sentry for cross-stack application error tracking
- AWS SES for transactional email (gap-fill)
- `hypothesis` for property-based unit testing of graph/canary/scoring invariants
- Playwright for E2E testing of dashboard demo flows
- `just` for cross-platform dev workflow task running
- Custom pytest harness (not DeepEval/Ragas) for AI evaluation suites
- Staged graph-database strategy (NetworkX → recursive CTEs → Neo4j/Neptune)
- `testcontainers-python` for real Postgres/Redis in integration tests

**Proceed to:** `16_Development_Roadmap_15_Months.md` — sequencing all Module A–F implementation milestones, research deliverable checkpoints, evaluation suite build-out, design-partner onboarding (G2), and SIH demo preparation across 15 months, with the Month-1 blocking spikes from Section 15.2 as the first sprint's explicit deliverables.

---

*Document authored by: Principal Software Architect · Principal AI Engineer · DevOps Architect · Cybersecurity Architect · Startup CTO · Cloud Solutions Architect · VC Technical Due Diligence Reviewer*

*Status: APPROVED — Official Technology Reference, PreFlight v1.0 (15-month build)*
