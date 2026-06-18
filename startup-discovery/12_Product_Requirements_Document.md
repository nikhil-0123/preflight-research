# 12_Product_Requirements_Document.md

# Product Requirements Document (PRD)

## Product Name

**PreFlight**

## Tagline

Know what your AI agent can be tricked into doing — before it goes live.

## One-Line Description

PreFlight is a permission-graph-aware attack-surface scanner for MCP-connected and tool-using AI agents: it statically maps every tool/data/credential an agent can reach, computes the highest-risk attack paths, and dynamically validates the most dangerous ones via sandboxed, LLM-driven exploit simulation — producing a prioritized, standards-mapped remediation report.

## Document Status

Approved — derived from `11_Final_Startup_Selection.md` (GO decision, Section 14)

## Audience

System architects, AI/ML engineers, backend/frontend engineers, security engineers, and the founding team (3 engineers) for the 15-month build.

---

# 1. Background and Strategic Context

PreFlight is the project selected in `11_Final_Startup_Selection.md`, repositioning the original "AI Agent Permission Simulation and Risk Analysis Platform" concept from `06_Finalist_Comparison.md` into a narrowly-scoped, defensible wedge: an **MCP & Agent Permission Attack-Surface Scanner**.

It exists at the intersection of three converging forces, established during research:

1. **Regulatory pressure** — NIST AI Agent Standards Initiative, AIUC-1 (Q2 2026, mandatory third-party/MCP access monitoring, control E009), and EU AI Act Article 55 (mandatory adversarial testing) are creating compliance demand for pre-deployment agent risk assessment.
2. **A documented, severe security gap** — MCP servers carry CVEs with CVSS 7.3–9.6 affecting 437,000+ installations; 43% of analyzed MCP servers are vulnerable to command injection; prompt-injection losses are estimated at $2.3B (+340% YoY) with only ~23% of sophisticated attempts currently detected.
3. **An open competitive niche** — authorization/identity vendors (Aembit, TrueFoundry) solve "who can call what" plumbing; red-teaming/benchmark vendors (Mindgard, PyRIT, AgentDojo-derived tools) solve "is this model generally vulnerable." **Nobody combines a permission-graph model of a specific deployment with LLM-driven dynamic simulation of what an attacker could actually achieve through it.**

---

# 2. Product Vision

**Vision statement:** Every AI agent deployment, before it goes to production, should be able to answer the question *"if an attacker controls the input to this agent, what is the worst thing they could make it do with the tools and data it has access to — and how likely is that to actually work?"* — with the same rigor and automation that SAST/DAST brought to traditional application security.

**Long-term positioning:** "Snyk for AI agent permissions" — start as a developer-facing scanner integrated into CI/CD, expand into continuous monitoring and an enterprise compliance platform (see Section 15, Roadmap).

---

# 3. Goals and Success Criteria

## 3.1 Business / Startup Goals

* G1: Ship a working CLI + web scanner that produces a credible, demo-able attack-path report from a real MCP server configuration within Month 6.
* G2: Achieve at least one design partner (open-source maintainer, AI startup, or university lab) using PreFlight against a real MCP deployment by Month 9.
* G3: Map all findings to at least 3 recognized standards/frameworks (NIST AI Agent Standards Initiative, AIUC-1, OWASP Agentic AI Top 10 / MITRE ATLAS) by Month 9, to support a compliance-driven sales narrative.
* G4: Produce at least one research paper / preprint on permission-graph-conditioned agent red-teaming by Month 12.
* G5: Have a CI/CD-integrated (GitHub Action) version with a free tier by Month 12, to begin building the adoption/data flywheel described in `11_Final_Startup_Selection.md` Section 11.

## 3.2 Technical Goals

* T1: Build a correct, extensible **permission graph model** covering MCP servers, tools, scopes, data resources, and credentials.
* T2: Implement **static attack-path analysis** (graph search for high-risk paths to "sensitive sink" nodes) that runs in seconds for realistic configurations (10–200 tools).
* T3: Implement a **sandboxed dynamic simulation engine** using LLM-driven attacker/victim/judge agents that can demonstrate end-to-end exploit chains against mock tool implementations.
* T4: Achieve a **risk scoring model** combining static graph topology with dynamic simulation outcomes, validated against a labeled set of known-vulnerable configurations.
* T5: Ensure the entire pipeline runs on the team's hardware (RTX 2050 / 24GB RAM / i5 H-series) for development, with cloud deployment (AWS) for the hosted SaaS demo.

## 3.3 Academic / Recruiter Goals

* A1: Final-year project demonstrating graph algorithms, LLM orchestration, sandboxed execution environments, full-stack development, and security domain expertise.
* A2: SIH-ready live demo (per `11_Final_Startup_Selection.md` Section 13).
* A3: At least one publishable research contribution on permission-graph-conditioned automated red-teaming.

## 3.4 Non-Goals (Explicit Out-of-Scope for MVP)

To prevent the scope creep that contributed to TaskAnchor's rejection, the following are **explicitly excluded** from the MVP and Phase 1:

* ❌ Continuous, always-on production runtime monitoring of live agents (this is Phase 2+, see Section 15.3).
* ❌ Cross-framework runtime instrumentation (LangChain/CrewAI/AutoGen agent tracing).
* ❌ A general-purpose LLM red-teaming benchmark/leaderboard (this was ThreatBench; rejected as a product — see Section 11 for how its corpora are reused).
* ❌ Execution of simulations against real production credentials, real production tool backends, or any system the user does not explicitly designate as a sandbox target.
* ❌ Full enterprise SIEM/SOC functionality, autonomous remediation/response actions, or multi-tenant enterprise RBAC (Phase 3+).
* ❌ Support for non-MCP agent frameworks in v1 (LangChain-native tools, OpenAI Assistants function-calling without MCP) — these are explicitly Phase 2 (Section 15.2).

---

# 4. Target Users and Personas

## 4.1 Primary Persona: AI Platform Engineer ("Priya")

* **Role:** Builds and maintains internal AI agent infrastructure at an AI-native startup or SaaS company.
* **Goal:** Wants a CI check that fails the build if a new agent config grants dangerous tool-combinations (e.g., "read CRM" + "send email" + "browse web" on the same agent without human-in-the-loop).
* **Pain today:** Permission grants are reviewed manually, ad hoc, or not at all. No tooling exists to reason about *combinations* of tool access.
* **Primary surface:** CLI + GitHub Action / CI integration.

## 4.2 Secondary Persona: AppSec / Security Engineer ("Marcus")

* **Role:** Security engineer responsible for reviewing AI/agent deployments as part of broader AppSec program.
* **Goal:** Wants a BloodHound-style visual attack-path graph across the org's agent fleet, prioritized by exploitability.
* **Pain today:** No equivalent of BloodHound/CSPM exists for agent tool permissions; security review of agents is manual and inconsistent.
* **Primary surface:** Web dashboard with graph visualization, exportable reports.

## 4.3 Tertiary Persona: Compliance Officer ("Anjali")

* **Role:** Responsible for demonstrating regulatory compliance (EU AI Act Article 55, internal AIUC-1-aligned policy).
* **Goal:** Wants an exportable report mapping findings to specific control IDs (e.g., AIUC-1 E009) as audit evidence.
* **Pain today:** No tooling produces standards-mapped evidence for agent deployments specifically.
* **Primary surface:** PDF/exportable compliance report.

## 4.4 Channel Persona: MSSP / Security Consultant ("Dev")

* **Role:** Runs security assessments for multiple client organizations.
* **Goal:** Wants a repeatable, white-label-able assessment tool to run against client MCP deployments as a billable service.
* **Pain today:** No existing tool category for "agent permission pentest" exists as a packaged offering.
* **Primary surface:** Multi-project CLI/web tool with report export (Phase 2+).

---

# 5. Core Use Cases / User Stories

| ID | As a... | I want to... | So that... | Priority |
|---|---|---|---|---|
| UC-1 | AI Platform Engineer | run `preflight scan ./mcp-config.json` locally | I can see my agent's permission graph and top risks before deploying | P0 |
| UC-2 | AI Platform Engineer | add PreFlight as a CI step | a PR is flagged/blocked if it introduces a high-risk tool combination | P0 |
| UC-3 | Security Engineer | view an interactive graph of all tools, scopes, and reachable data per agent | I can visually identify dangerous attack paths | P0 |
| UC-4 | Security Engineer | trigger a dynamic simulation against a flagged attack path | I can see proof (transcript) of whether the attack actually succeeds, not just a theoretical path | P0 |
| UC-5 | Compliance Officer | export a report mapping findings to AIUC-1 / NIST / OWASP Agentic control IDs | I can use it as audit evidence | P1 |
| UC-6 | AI Platform Engineer | get a per-finding remediation recommendation (e.g., "scope this tool to read-only", "require human approval before send_email") | I know exactly how to fix the issue | P0 |
| UC-7 | Security Engineer | configure custom "sensitive sink" definitions (e.g., "any tool that sends external email or HTTP requests is a sink") | the risk model reflects my org's specific risk tolerance | P1 |
| UC-8 | MSSP Consultant | run scans across multiple client projects with isolated reports | I can offer this as a service | P2 (Phase 2) |
| UC-9 | AI Platform Engineer | re-run a scan on a schedule and get alerted on permission *drift* (new tool grants since last scan) | I have lightweight continuous assurance without full runtime monitoring | P2 (Phase 2) |
| UC-10 | Researcher | export anonymized scan results + simulation transcripts in a structured format | I can use them for the research publication described in Section 12 of `11_Final_Startup_Selection.md` | P1 |

---

# 6. Functional Requirements

## 6.1 Module A — Configuration Ingestion & Permission Graph Builder

### A1. Input Sources (MVP)

* **FR-A1.1:** Accept MCP server configuration files (JSON, per the MCP specification — server manifests, tool definitions, declared scopes).
* **FR-A1.2:** Accept agent configuration files that declare which MCP servers/tools an agent instance is connected to (support common formats: Claude Desktop-style `mcp.json`, generic JSON schema defined by PreFlight for frameworks that don't natively export this).
* **FR-A1.3:** Support a PreFlight-native YAML/JSON schema as a fallback/manual-entry format for agents whose framework doesn't expose a machine-readable config (documented schema, with a CLI wizard for manual entry — `preflight init`).
* **FR-A1.4:** Validate inputs against expected schema; produce clear, actionable error messages for malformed configs.

### A2. Permission Graph Model

* **FR-A2.1:** Construct a directed graph with the following node types:
  * `Agent` — a specific agent instance/deployment.
  * `MCPServer` — a connected MCP server.
  * `Tool` — an individual tool exposed by an MCP server, with its declared scope(s) per the MCP 2026 tool-level RBAC spec.
  * `DataResource` — a logical data asset a tool can read/write (e.g., "CRM contacts", "filesystem:/home", "email outbox") — inferred from tool descriptions/schemas where possible, and explicitly declarable.
  * `Credential` — a credential/secret a tool uses to authenticate to a backend.
  * `ExternalSink` — any node representing data leaving the trust boundary (e.g., "send email", "HTTP POST to arbitrary URL", "write to public storage").
* **FR-A2.2:** Construct edges representing:
  * `Agent -> Tool` (agent can invoke tool)
  * `Tool -> DataResource` (read/write access, with read/write distinguished)
  * `Tool -> Credential` (tool uses credential)
  * `Tool -> ExternalSink` (tool can exfiltrate/act externally)
  * `DataResource -> DataResource` (data flow relationships, where inferable — Phase 2)
* **FR-A2.3:** Each node/edge carries metadata: sensitivity level (user-tagged or inferred), scope/permission level (read/write/admin), source (which config file/line it came from, for traceability in reports).
* **FR-A2.4:** Support a tagging system so users can mark specific `DataResource` or `ExternalSink` nodes as "sensitive" (FR-A7 below covers configurability).
* **FR-A2.5:** Graph must be serializable (export to JSON/GraphML) for use by Module B, Module C, and the frontend visualization (Module D).

### A3. Tool Description Analysis (LLM-assisted)

* **FR-A3.1:** For tools whose declared scope/description does not clearly indicate sensitivity (read vs. write, internal vs. external), use an LLM-based classifier to infer: (a) data sensitivity category, (b) read/write/execute classification, (c) whether the tool constitutes an `ExternalSink`.
* **FR-A3.2:** All LLM-inferred classifications must be flagged as "inferred" in the UI/report and be user-editable/overridable.
* **FR-A3.3:** Maintain a cached, versioned classification database for common/popular open-source MCP servers (seeded from public MCP server registries) to avoid redundant LLM calls.

---

## 6.2 Module B — Static Attack Path Analysis Engine

### B1. Sensitive Sink Definition

* **FR-B1.1:** Provide a default library of "sensitive sink" patterns: external communication tools (email, Slack/webhook, arbitrary HTTP), data export tools (file write to shared/public locations), destructive operations (delete, drop table), and privilege-escalation-relevant tools (credential read, config write).
* **FR-B1.2:** Allow users to define custom sink rules (FR per UC-7) via config file (e.g., regex/semantic match on tool names/descriptions, or manual graph annotation).

### B2. Attack Path Computation

* **FR-B2.1:** For each `Agent` node, compute all paths from `Agent` to any `ExternalSink` or tagged-sensitive `DataResource`/`Credential` node, up to a configurable max depth (default depth: 4).
* **FR-B2.2:** For each path, compute a **Static Risk Score** based on: (a) number of hops, (b) sensitivity of intermediate `DataResource` nodes traversed, (c) read/write/admin level of edges traversed, (d) whether the path crosses a trust boundary (e.g., from an `ExternalSink`-adjacent input tool to a sensitive sink — i.e., "input tool -> ... -> output tool" chains, which represent classic prompt-injection-to-exfiltration patterns).
* **FR-B2.3:** Rank all discovered paths by Static Risk Score; surface the top N (configurable, default 10) as "candidate attack paths" for dynamic simulation (Module C).
* **FR-B2.4:** Detect and flag specific known-dangerous *patterns* explicitly (pattern library, extensible):
  * "Lethal trifecta": an agent with (1) access to untrusted/external input, (2) access to sensitive private data, and (3) ability to communicate externally — flagged regardless of path length, as this is the canonical prompt-injection-to-exfiltration setup.
  * "Confused deputy": a tool that acts with higher privilege than the calling agent's nominal role requires.
  * "Credential reuse": multiple tools sharing the same underlying credential, creating cross-tool blast radius.

### B3. Graph Algorithm Requirements

* **FR-B3.1:** Path-finding must use efficient graph algorithms (e.g., weighted shortest-path / k-shortest-paths, Dijkstra or Yen's algorithm variants) — must complete within seconds for graphs up to ~1,000 nodes / ~5,000 edges (realistic upper bound for an enterprise agent fleet config).
* **FR-B3.2:** Must support incremental re-computation (for CI use — only re-analyze changed portions of the graph where feasible, Phase 2 optimization).

---

## 6.3 Module C — Dynamic Simulation Engine (LLM-Driven)

This is PreFlight's core technical differentiator and must be designed with safety as a first-class constraint (see Section 18, Safety & Sandbox Requirements).

### C1. Simulation Architecture

* **FR-C1.1:** For each candidate attack path selected by Module B, instantiate a **sandboxed simulation environment** consisting of:
  * A **Victim Agent** — an LLM configured with the system prompt, tool definitions, and tool access matching the real agent under test (the model itself can be configurable: the user's actual model, or a default reference model).
  * **Mock Tool Implementations** — sandboxed stand-ins for each real tool in the path, returning realistic mock data (see C2).
  * An **Attacker Agent** — an LLM tasked with crafting an adversarial input (the only input channel available to a real-world attacker — e.g., content of an email, a webpage, a document the Victim Agent will process) designed to manipulate the Victim Agent into traversing the candidate attack path.
  * A **Judge** — an LLM (or rule-based check, where possible) that evaluates the resulting transcript to determine whether the attack path was successfully traversed (e.g., "did sensitive data X appear in the output of ExternalSink tool Y?").

### C2. Mock Tool Implementations

* **FR-C2.1:** Maintain a library of mock implementations for common MCP tool categories (filesystem, email/messaging, HTTP/web-fetch, database/CRM query, calendar) that return realistic synthetic data (e.g., synthetic "customer records" with marked canary values for exfiltration detection).
* **FR-C2.2:** Mock tools must inject **canary tokens** (unique, traceable synthetic values) into sensitive `DataResource` mocks so the Judge can deterministically detect exfiltration via simple string matching, reducing reliance on LLM-judge accuracy for the core success signal.
* **FR-C2.3:** Where a real (non-destructive, sandboxed) tool implementation is available and the user explicitly opts in (e.g., a disposable test database), allow substitution of real tool implementations for higher-fidelity simulation — but never by default (see Section 18).

### C3. Attacker Agent Behavior

* **FR-C3.1:** The Attacker Agent must be seeded with: (a) the specific attack path being tested (which tools/data/sinks are involved), (b) the injection surface (which tool's *output* the Victim Agent will read — i.e., where the attacker's payload is "placed"), and (c) a library of known prompt-injection techniques as a starting point.
* **FR-C3.2:** Seed the Attacker Agent's technique library from the open academic corpora identified in `11_Final_Startup_Selection.md` (AgentDojo, InjecAgent, ASB, Agent-SafetyBench attack taxonomies) — converting ThreatBench's surveyed material into PreFlight's R&D asset, per the GO decision.
* **FR-C3.3:** Support iterative/multi-turn attack refinement: if a first attempt fails, the Attacker Agent may observe the Victim Agent's response and refine its payload, up to a configurable max attempts (default: 3) per path, to balance thoroughness against cost.

### C4. Judge & Outcome Recording

* **FR-C4.1:** For each simulation run, record a structured outcome: `SUCCESS` / `PARTIAL` / `FAILURE`, with supporting evidence (canary token match, transcript excerpt, judge reasoning).
* **FR-C4.2:** `PARTIAL` covers cases where the Victim Agent took steps toward the attack path but did not complete it (e.g., it called the sensitive-data tool but refused to call the exfiltration tool) — these are valuable findings (near-misses) and must be surfaced distinctly.
* **FR-C4.3:** Full simulation transcripts (all agent turns, tool calls, and responses) must be stored and made available in the report for human review (FR-D, Module D).

### C5. Dynamic Risk Score

* **FR-C5.1:** Combine the Static Risk Score (Module B) with the simulation outcome to produce a final **Composite Risk Score** per attack path: e.g., `SUCCESS` outcomes significantly elevate the score regardless of static topology; `FAILURE` outcomes reduce but do not zero out the score (a failed simulation with one model/prompt does not prove the path is safe in general).
* **FR-C5.2:** The scoring model and its weighting must be documented, versioned, and configurable — this is itself a research artifact (see `11_Final_Startup_Selection.md`, Section 12, Research Question 2).

---

## 6.4 Module D — Reporting, Dashboard, and Remediation

### D1. Web Dashboard (React)

* **FR-D1.1:** Interactive graph visualization of the full permission graph (per agent/project), with attack paths highlighted/color-coded by Composite Risk Score.
* **FR-D1.2:** Drill-down view per attack path: graph segment, Static Risk Score breakdown, simulation status, and (if run) full simulation transcript.
* **FR-D1.3:** Findings list view: sortable/filterable table of all findings by severity, pattern type (FR-B2.4), and standards mapping.
* **FR-D1.4:** Diff view (Phase 2): compare two scans of the same project to highlight permission drift (new tools, new data access, new sinks) — feature flag, scoped per UC-9.

### D2. Remediation Recommendations

* **FR-D2.1:** Every finding must include a concrete, actionable remediation recommendation. Recommendation types include: scope reduction (e.g., "change tool X from read-write to read-only"), human-in-the-loop gating ("require approval before calling tool Y"), tool segregation ("split tool Z's credential from tool W's"), and input sanitization pointers (where the attack relies on a specific injection vector).
* **FR-D2.2:** Where feasible, generate a suggested config diff (e.g., a patch to the MCP server manifest or agent config) implementing the recommendation.

### D3. Standards Mapping

* **FR-D3.1:** Every finding pattern (FR-B2.4) and every Composite Risk Score tier must be mapped to relevant control IDs in: NIST AI Agent Standards Initiative guidance, AIUC-1 (specifically control E009 — third-party/MCP access monitoring — and related tool-authorization controls), OWASP Agentic AI Top 10 (or equivalent OWASP Agentic Security project taxonomy), and MITRE ATLAS tactic/technique IDs.
* **FR-D3.2:** This mapping table must be maintained as a versioned, structured data file (not hardcoded in templates) so it can be updated as standards evolve without code changes.

### D4. Report Export

* **FR-D4.1:** Export a full report (findings, graph, remediation, standards mapping) as PDF and as structured JSON (for programmatic consumption / CI integration).
* **FR-D4.2:** Export anonymized scan results + simulation transcripts in a structured research dataset format (per UC-10), with PII/secrets scrubbing.

---

## 6.5 Module E — CLI and CI/CD Integration

* **FR-E1:** `preflight scan <path>` — run a full scan (Modules A–C) against a local config directory, output summary to terminal + write full report to disk.
* **FR-E2:** `preflight scan --static-only` — skip Module C (dynamic simulation) for fast CI runs; run full simulation on a schedule or on-demand.
* **FR-E3:** `preflight ci` — designed for CI pipelines: exits non-zero if any finding exceeds a configurable Composite Risk Score threshold (for PR-blocking).
* **FR-E4:** GitHub Action wrapper around `preflight ci`, publishing results as a PR check/annotation.
* **FR-E5:** All CLI output and exit codes must be machine-parseable (JSON output mode) in addition to human-readable terminal output.

---

## 6.6 Module F — Backend API and Project Management

* **FR-F1:** FastAPI backend exposing REST endpoints for: project CRUD, triggering scans, retrieving scan results/history, retrieving graph data, triggering individual simulations, retrieving reports.
* **FR-F2:** Authentication (initially: simple email/password or OAuth via a managed provider; multi-tenant org/role model deferred to Phase 2 per Non-Goals).
* **FR-F3:** Persistent storage of: project configs, graph snapshots (per scan), findings, simulation transcripts, and historical scan metadata (for drift detection, Phase 2).
* **FR-F4:** Asynchronous task queue for scan/simulation jobs (simulations involve multiple LLM calls and may take minutes) — jobs must be cancellable and progress-trackable.

---

# 7. Non-Functional Requirements

| ID | Category | Requirement |
|---|---|---|
| NFR-1 | Performance | Static analysis (Modules A+B) for a graph of up to 1,000 nodes completes in < 10 seconds. |
| NFR-2 | Performance | A single dynamic simulation (Module C, one attack path, default max-attempts) completes in < 2 minutes wall-clock (dominated by LLM API latency). |
| NFR-3 | Cost control | Default scan configuration (static + top-10 dynamic simulations) must cost less than a defined per-scan LLM API budget (target: configurable, default ≤ $1–2 per scan using a cost-efficient model tier); local/open-weight model option (via Ollama) must be supported for development and cost-sensitive users. |
| NFR-4 | Scalability | Backend architecture must support horizontal scaling of the simulation worker pool independently from the API layer (queue-based, e.g., Celery/RQ + Redis or AWS SQS). |
| NFR-5 | Security | No scan or simulation may execute against real production credentials or live external endpoints by default (see Section 18). |
| NFR-6 | Security | All uploaded configuration files are treated as potentially sensitive; encrypted at rest; access-controlled per project. |
| NFR-7 | Reliability | Scan/simulation jobs must be idempotent and resumable on worker failure. |
| NFR-8 | Portability | Core engine (Modules A–C) must run as a standalone library/CLI independent of the web backend, for open-source distribution and the adoption flywheel (per `11_Final_Startup_Selection.md` Section 11, Monetization Path step 1). |
| NFR-9 | Observability | All LLM calls (attacker, victim, judge, classifier) must be logged with prompts/responses (for debugging, research export, and cost tracking), with secrets/PII redaction. |
| NFR-10 | Extensibility | Graph schema, sink pattern library, mock tool library, and standards-mapping table must all be data-driven/config-defined, not hardcoded, to support rapid extension as MCP ecosystem evolves. |

---

# 8. Data Model (Core Entities)

```
Project
  - id, name, owner, created_at
  - configs: [ConfigFile]

ConfigFile
  - id, project_id, type (mcp_server | agent | preflight_native)
  - raw_content, parsed_at

Scan
  - id, project_id, triggered_by, created_at, status
  - graph_snapshot_id

GraphSnapshot
  - id, scan_id
  - nodes: [Node]   # Agent, MCPServer, Tool, DataResource, Credential, ExternalSink
  - edges: [Edge]

Finding
  - id, scan_id, attack_path (ordered list of node/edge ids)
  - pattern_type (lethal_trifecta | confused_deputy | credential_reuse | generic_path | ...)
  - static_risk_score
  - simulation: Simulation (nullable)
  - composite_risk_score
  - remediation: RemediationRecommendation
  - standards_mappings: [StandardsMapping]

Simulation
  - id, finding_id
  - attacker_transcript, victim_transcript, judge_output
  - outcome (success | partial | failure)
  - attempts: int
  - cost_usd, duration_ms

RemediationRecommendation
  - id, finding_id
  - type (scope_reduction | hitl_gate | tool_segregation | input_sanitization)
  - description, suggested_config_patch (nullable)

StandardsMapping
  - id, finding_pattern_type
  - framework (NIST_AGENT | AIUC1 | OWASP_AGENTIC | MITRE_ATLAS)
  - control_id, description
```

---

# 9. AI / LLM Architecture Requirements

| Component | Role | Model Tier Guidance |
|---|---|---|
| Tool Classifier (FR-A3.1) | Classifies ambiguous tool descriptions for sensitivity/read-write/sink status | Small/cheap model (e.g., 7–13B local via Ollama, or cheapest hosted tier); cached aggressively |
| Attacker Agent (Module C) | Crafts adversarial inputs to traverse a candidate attack path | Mid-tier hosted model recommended for quality of injection crafting; local model option for dev/cost |
| Victim Agent (Module C) | Simulates the real agent under test | Configurable — ideally matches the model the real deployment uses, since vulnerability is model-dependent |
| Judge (Module C) | Evaluates simulation transcripts for success | Mid-tier hosted model + rule-based canary-token check as primary signal (FR-C2.2) to minimize reliance on LLM judgment alone |

**Key architectural principle:** the canary-token mechanism (FR-C2.2) is the **primary, deterministic** success signal; the LLM Judge is a **secondary, explanatory** layer. This directly addresses the "judge reliability" research question (Section 12 of `11_Final_Startup_Selection.md`) by not over-relying on LLM-as-judge for ground truth.

**Hardware mapping:** Given the RTX 2050/24GB constraint, local model usage (via Ollama, quantized 7–13B models) is viable for the Tool Classifier and for development/testing of the Attacker/Victim/Judge pipeline, but the production hosted SaaS and any high-fidelity demo (SIH, design partners) should use hosted API models for the Attacker/Judge roles to maximize simulation quality — cost-managed via NFR-3.

---

# 10. Standards Mapping Requirements (Detail)

The standards-mapping table (FR-D3.1/D3.2) is a first-class product asset and a key differentiator/procurement narrative. Initial mapping targets:

* **AIUC-1, Control E009** (third-party/MCP access monitoring) — direct mapping for the entire permission-graph + scan output.
* **AIUC-1** tool-level authorization & I/O validation controls — mapping for Module B's "confused deputy" and "credential reuse" patterns.
* **NIST AI Agent Standards Initiative** — mapping for identity/authorization-related findings.
* **OWASP Agentic AI Security project taxonomy** — mapping for the "lethal trifecta" pattern (prompt-injection-to-exfiltration) and general tool-abuse findings.
* **MITRE ATLAS** — mapping relevant tactics/techniques (e.g., resource development, exfiltration via ML-enabled product) to each finding pattern.

This table must be designed for **versioned updates** as these standards (several of which are actively evolving, per the research in `11_Final_Startup_Selection.md` Section 5.2) mature.

---

# 11. Dataset Strategy

| Source | Use |
|---|---|
| Public MCP server registries / open-source MCP server repos | Seed the Tool Classifier cache (FR-A3.3); provide realistic graph-construction test fixtures |
| Documented MCP CVEs (CVSS 7.3–9.6 set) | Validate that PreFlight's pattern library (FR-B2.4) and simulations can detect known-real vulnerabilities — core validation benchmark |
| AgentDojo, InjecAgent, ASB, Agent-SafetyBench attack taxonomies | Seed library for the Attacker Agent (FR-C3.2) |
| Synthetic canary-tagged mock data (FR-C2.2) | Generated in-house; no external dependency |
| Anonymized user scan results (opt-in) | Long-term data moat (per `11_Final_Startup_Selection.md` Section 11); also research dataset (UC-10) |

**No production customer telemetry is required for the MVP** — this is the central feasibility advantage over the rejected TaskAnchor approach (per Section 9/10 of `11_Final_Startup_Selection.md`).

---

# 12. Success Metrics / KPIs

| Metric | Target (by Month 12) |
|---|---|
| Known-CVE detection rate | PreFlight's pattern library + simulation correctly flags ≥ 80% of a curated set of documented MCP CVEs as high-risk findings |
| Static analysis performance | < 10s for 1,000-node graphs (NFR-1) |
| Simulation success-rate calibration | Composite Risk Score correlates with simulation outcome in a held-out validation set (research deliverable) |
| CI integration adoption | ≥ 1 external design partner running `preflight ci` in their pipeline |
| Standards coverage | 100% of finding patterns mapped to ≥ 1 standard each |
| Cost per scan | Within NFR-3 budget for default config |

---

# 13. Risks and Mitigations (Product-Level)

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Mock tool fidelity gap — simulations don't reflect real backends | Medium | High | Build mock library from real open-source MCP server schemas (FR-C2.1); offer opt-in real-tool substitution in safe sandboxes (FR-C2.3) |
| LLM Judge unreliability | Medium | Medium | Canary-token deterministic signal as primary (FR-C2.2/C4.1) |
| LLM API cost overruns for simulation | Medium | Medium | Local model option, attempt caps (FR-C3.3), top-N path selection (FR-B2.3), cost tracking (NFR-9) |
| MCP spec evolution breaks parsers | Medium | Medium | Versioned schema support (FR-A1.4), maintained as MCP spec evolves |
| Category crowding (per `11_Final_Startup_Selection.md` Section 5.5) | Medium | Medium | Narrow positioning + standards-alignment differentiation (Section 10) |
| Recurring-revenue durability for point-in-time scans | Medium | Medium | CI integration (Module E) + drift detection (Phase 2, Section 15.3) shift toward recurring value |

---

# 14. Open Questions / Assumptions for Architecture Phase

1. Graph storage: in-memory/NetworkX with serialized snapshots (simpler, sufficient for MVP scale) vs. a graph database (Neo4j) for the hosted product — needs decision in `13_System_Architecture.md`.
2. Which hosted LLM provider(s) for Attacker/Judge roles, and how multi-provider abstraction is handled (cost/quality tradeoffs) — to be detailed in `14_AI_Architecture.md`.
3. Sandbox execution technology for mock tools and (opt-in) real-tool substitution — containers (Docker) per simulation run vs. lighter-weight process isolation — to be detailed in `13_System_Architecture.md` and Section 18 safety design.
4. Exact initial set of supported "agent config" formats beyond MCP-native (FR-A1.2/A1.3) — to be prioritized based on observed real-world configs from design partners.

---

# 15. Roadmap Alignment (Phasing Summary)

This PRD covers **Phase 1 (MVP, Months 1–9 approx.)**. Full roadmap detail is deferred to `16_Development_Roadmap_15_Months.md`, but phase boundaries are summarized here to anchor scope decisions:

## 15.1 Phase 1 — MVP (this PRD)

Modules A–F as specified above: permission graph builder, static attack path analysis, dynamic simulation engine, reporting/dashboard, CLI/CI integration, standards mapping.

## 15.2 Phase 2 — Expansion (post-MVP)

* Additional agent framework support beyond MCP-native configs (LangChain, CrewAI, OpenAI Agents SDK tool definitions).
* Permission-drift detection across scheduled re-scans (UC-9) — the controlled, scoped slice of TaskAnchor's original vision (per `11_Final_Startup_Selection.md` Section 7, Disposition).
* Multi-project/multi-tenant support for MSSP use case (UC-8).
* Incremental graph re-computation for large configs (FR-B3.2).

## 15.3 Phase 3 — Platform (post-15-months / startup continuation)

* Continuous monitoring integrations (lightweight runtime hooks, *not* full cross-framework instrumentation).
* Enterprise RBAC, SSO, audit logging.
* Marketplace/community-contributed mock tool library and pattern library.

---

# 16. Glossary

| Term | Definition |
|---|---|
| MCP | Model Context Protocol — standard for connecting AI agents to external tools/data sources |
| Permission Graph | PreFlight's internal graph model of an agent's reachable tools, data, credentials, and external sinks |
| Attack Path | A path in the permission graph from an agent to a sensitive sink/data resource |
| Sensitive Sink | A node representing data/action leaving the trust boundary (external comms, destructive ops, etc.) |
| Lethal Trifecta | Pattern: agent has (1) untrusted input access, (2) sensitive data access, (3) external communication ability |
| Canary Token | A unique synthetic value injected into mock sensitive data to deterministically detect exfiltration in simulations |
| Composite Risk Score | Combined score from static graph analysis (Module B) and dynamic simulation outcome (Module C) |

---

# 17. Approval

| Role | Status |
|---|---|
| Product / Startup CTO sign-off | Approved per `11_Final_Startup_Selection.md`, Section 14 (GO) |
| Next Document | `13_System_Architecture.md` |

---

# 18. Safety & Sandbox Constraints (Non-Negotiable)

These constraints are carried forward verbatim from the Go/No-Go conditions in `11_Final_Startup_Selection.md`, Section 14, item 4, and apply to all phases:

1. The Dynamic Simulation Engine (Module C) **must never** execute against real production systems, real production credentials, or any third-party system without explicit, documented, per-target user consent.
2. Default behavior for all simulations is **fully sandboxed mock tools** (FR-C2.1–C2.3).
3. Any opt-in "real tool substitution" (FR-C2.3) must require an explicit, separate confirmation step and must default to disposable/test-only resources (e.g., a throwaway test database, a sandboxed email account).
4. All generated adversarial payloads (Attacker Agent outputs) are stored and logged but **never automatically transmitted** to any system outside the simulation sandbox.
