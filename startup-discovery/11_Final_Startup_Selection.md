# 11_Final_Startup_Selection.md

# Final Startup Selection — Official Decision Record

## Project

AI Startup Discovery Lab — Phase 3: Final Startup Selection

## Status

**SUPERSEDES** `07_Selected_Startup.md` and `10_Go_NoGo_Decision.md`

## Final Selected Project

**PreFlight — MCP & Agent Permission Attack-Surface Scanner**

---

# 1. Executive Summary

This document is the official, final decision record for the AI Startup Discovery Lab. It closes out the opportunity discovery and validation process that began in `01_Project_Framework.md` and supersedes the previously approved selection of **TaskAnchor** recorded in `07_Selected_Startup.md` and `10_Go_NoGo_Decision.md`.

Phase 1 and Phase 2 correctly identified **AI Agent Security & Runtime Governance** as the strongest opportunity category for a 3-person AI engineering team, and correctly narrowed the field to three finalists: **PreFlight**, **ThreatBench**, and **TaskAnchor**. TaskAnchor was initially selected on the strength of its market size, recruiter value, and startup ceiling, despite already carrying the lowest technical feasibility score (7.0/10) and a "High" technical complexity risk rating.

Phase 3 was triggered by a deliberately adversarial re-evaluation panel whose mandate was to challenge — not confirm — the prior decision. The panel conducted fresh, dated competitive intelligence research (June 2026) and discovered that the competitive landscape underlying the original TaskAnchor decision had changed materially in the ~12 months since it was made. Specifically, the core differentiator claimed for TaskAnchor ("goal drift detection" as a unique capability) is now a **shipping, marketed feature** across multiple AI agent security and observability platforms that have collectively raised billions of dollars in funding.

Re-scoring all three finalists against the original 15-criterion framework, using updated evidence, produced a different result:

* **ThreatBench** — Rejected (reconfirmed). The benchmark space has gone from "wide open" to "saturated" by both academic labs and commercial tooling.
* **TaskAnchor** — Rejected (reversal). Its core moat claim is now commoditized, its technical feasibility gap has widened against better-funded competitors, and its dataset problem remains unsolved.
* **PreFlight** — Selected (reversal). Repositioned as a **narrow, defensible wedge** — an MCP/agent permission attack-surface scanner — PreFlight is the only finalist that simultaneously satisfies real customer pain, genuine differentiation, feasibility for a 3-person team on consumer hardware, and dataset availability without requiring production telemetry.

**Final Decision: GO — Proceed with PreFlight (repositioned) as the flagship project.**

---

# 2. Opportunity Discovery Journey

## Phase 1 — Framework and Constraints (`01_Project_Framework.md`)

Established the team profile (3 members, 15 months, RTX 2050 / 24GB RAM / i5 H-series), existing skills (Python, PyTorch, FastAPI, React, SQL, AWS basics), preferred domains (AI, Cybersecurity, Cloud, Data Engineering), and the evaluation framework spanning business, technical, practical, and career metrics. Critically, this phase also set the **rejection criteria**: no generic student projects, no YouTube clones, no inaccessible datasets, no enterprise-scale budget dependencies.

## Phase 2 — Opportunity Space Research (`02_Opportunity_Spaces.md`)

Eight opportunity spaces were identified and researched:

1. AI Agent Security & Runtime Governance
2. AI-Native Software Supply Chain Security
3. Shadow AI Governance
4. OT/ICS Critical Infrastructure Security
5. Edge AI MLOps
6. Digital Twin + Reinforcement Learning
7. Non-Human Identity Security
8. Post-Quantum Readiness Platforms

The strongest convergence of innovation, startup potential, research novelty, recruiter value, and feasibility was found in spaces 1–3.

## Phase 3 — Top 5 Ranking (`03_Top5_Opportunities.md`)

AI Agent Security & Runtime Governance ranked #1 overall (9.3/10), ahead of AI-Native Software Supply Chain Security (8.9/10), Shadow AI Governance (8.7/10), OT/ICS Critical Infrastructure Security (8.2/10), and Edge AI MLOps (8.0/10). This category became the focus of all subsequent research.

## Phase 4 — Rejected Opportunities (`04_Rejected_Opportunities.md`)

Three concepts were explicitly rejected before reaching the finalist stage:

* **Autonomous SOC / AI SOC Co-Pilot** — rejected due to an extremely crowded market (CrowdStrike, Microsoft, Google, Palo Alto, Splunk, and others) and infeasible data requirements for a student team.
* **Non-Human Identity Security (standalone)** — rejected due to rapid market consolidation via acquisition (Cisco/Astrix, CrowdStrike/SGNL, Cyera/Otterize) and enterprise-integration dependencies.
* **Post-Quantum Cryptography Readiness** — rejected due to skill misalignment (cryptography vs. AI/ML) and reduced AI relevance.

## Phase 5 — Final Findings (`05_Final_Findings.md`)

Confirmed AI Agent Security & Runtime Governance as the winning category and proposed five potential startup themes: Agent Runtime Security Platform, Agent Action Audit System, AI Agent Threat Detection Engine, Agent Identity Governance Platform, and MCP Security Platform. These themes seeded the three Phase 6 finalists.

---

# 3. Initial Selection of TaskAnchor

## Finalist Comparison (`06_Finalist_Comparison.md`)

Three concrete startup concepts were evaluated head-to-head:

| Metric | PreFlight | ThreatBench | TaskAnchor |
|---|---|---|---|
| Market Opportunity | 9.5 | 8.0 | 10.0 |
| Startup Potential | 9.0 | 7.0 | 10.0 |
| Research Potential | 8.5 | 10.0 | 9.0 |
| Recruiter Value | 9.5 | 9.0 | 10.0 |
| **Technical Feasibility** | **8.0** | **8.5** | **7.0** |
| SIH Potential | 7.5 | 8.5 | 8.0 |

TaskAnchor was ranked #1 primarily on Market Opportunity, Startup Potential, and Recruiter Value — but it already carried the **lowest Technical Feasibility score of the three finalists**, a fact that was acknowledged but not weighted heavily enough at the time.

## Selection and Validation (`07_Selected_Startup.md`, `08_Market_Validation.md`)

TaskAnchor — "Runtime Governance and Security for Autonomous AI Agents" — was selected. Market validation concluded "GO" with High market need, High customer pain, Excellent market timing, but only **Moderate** technical feasibility.

## Competitor Analysis (`09_Competitor_Analysis.md`)

Competitive intensity was assessed as **Medium**. The claimed competitive moat rested explicitly on a feature comparison table asserting:

| Feature | TaskAnchor | Lakera | Protect AI | Langfuse |
|---|---|---|---|---|
| Goal Drift Detection | **Yes** | No | No | No |

This single row — "we do goal drift detection and nobody else does" — was the load-bearing differentiation claim for the entire startup thesis.

## Go/No-Go Decision (`10_Go_NoGo_Decision.md`)

Final verdict: **GO**, overall score 8.9/10, with Technical Complexity flagged as the only **High**-level risk (all other risks rated Medium). The mitigation strategy was to start with rule-based monitoring and add ML incrementally. Startup success probability was estimated at 15–25%.

**This is the decision Phase 3 was tasked with re-testing.**

---

# 4. Re-evaluation Process

## Trigger

The Phase 3 panel was convened with an explicit anti-confirmation mandate: *"Your role is not to support ideas automatically. Your job is to kill weak ideas."* Given that `10_Go_NoGo_Decision.md` already flagged Technical Complexity as the single highest risk factor for TaskAnchor, and that the entire competitive moat rested on one feature-comparison row, this was identified as the highest-priority assumption to stress-test.

## Panel Composition

* YC Partner
* Sequoia Partner
* Cybersecurity Startup Founder
* Principal AI Engineer
* Principal Security Architect
* Smart India Hackathon Grand Finale Judge
* Venture Capital Technical Due Diligence Expert

## Methodology

1. Re-apply the original 15-criterion evaluation framework (real customer problem, willingness to pay, competitors, why competitors haven't solved it, technical risks, business risks, datasets, custom data collection, research potential, recruiter value, startup potential, team difficulty, SIH potential, MVP probability, startup probability) to all three finalists.
2. Conduct dated, sourced competitive intelligence research (June 2026) across four sub-domains: (a) agentic AI security funding/M&A, (b) MCP security and regulatory standards, (c) AI agent runtime observability/governance platforms, and (d) AI agent red-teaming/benchmark research.
3. Re-score all three finalists against the updated evidence.
4. Force a single winner; reject the other two with explicit reasoning.

---

# 5. Updated Competitive Landscape Analysis

## 5.1 Agentic AI Security Funding & M&A (Q1–Q2 2026)

The agentic AI security category has gone from "emerging" to one of the hottest sub-sectors in all of cybersecurity venture capital:

* The top startups building agentic AI defenses have collectively raised approximately **$3.6 billion**, against a backdrop of roughly **$96 billion** in cybersecurity M&A across ~400 transactions (Momentum Cyber 2025 Almanac).
* A single week around RSAC 2026 produced **$392 million** in new agentic AI security funding announcements across six companies.
* **MCP security** specifically — the exact protocol layer PreFlight targets — has attracted **~$40 million** in disclosed dedicated funding and was explicitly called out as "the new category" at RSAC 2026.
* Named rounds directly adjacent to TaskAnchor's positioning: WitnessAI ($58M in January 2026, $85M+ total), Noma Security ($100M for "AI agent hardening"), Oasis Security ($120M Series B for "agentic access governance"), Trent AI (new $13M-seed London entrant, April 2026), and Cyera ($400M Series F, $1.7B total funding, $9B valuation).

## 5.2 MCP Security & Regulatory Landscape

This is the sub-domain most relevant to PreFlight, and it has shifted from "informal best practice" to "active regulatory mandate":

* **NIST AI Agent Standards Initiative** — formally launched to regulate autonomous system security, with internal NIST red-team research showing novel attack strategies against AI agents succeed roughly **81%** of the time.
* **NSA Cybersecurity Information Sheet (CSI)** — explicitly warns that MCP adoption creates new attack surfaces that traditional security tooling cannot address.
* **AIUC-1 standard, Q2 2026 update** — extended its requirements to MCP specifically: tool-level authorization and input/output validation now apply to MCP tool calls, logging must capture MCP server-level metadata (tool name, input parameters), and **third-party access monitoring is now a mandatory requirement (E009)** — covering MCP servers, third-party agents, and plugin registries discovered dynamically at runtime.
* **EU AI Act Article 55** — requires documented adversarial testing for relevant AI systems.
* **MCP specification 2026 update** — introduced incremental scope consent and requires tool-level RBAC, where each individual tool within an MCP server carries its own authorization requirements evaluated per invocation — a meaningfully harder problem than server-level access control.
* **Real-world severity data** — MCP-related CVEs with CVSS scores of 7.3–9.6 have been identified affecting over 437,000 installations; 43% of analyzed MCP servers were found vulnerable to command injection.

**Net effect:** Regulators and standards bodies are now actively *creating demand* for exactly the kind of pre-deployment, permission-aware risk assessment PreFlight proposes — without yet producing a dominant incumbent in this specific niche.

## 5.3 AI Agent Runtime Governance & Observability Landscape

This is the sub-domain most relevant to TaskAnchor, and it is now **severely crowded**:

* Security-focused runtime governance players: WitnessAI, Noma Security, Pillar Security, AIM Security, Operant AI, Lakera, Protect AI, HiddenLayer, CalypsoAI, Trent AI.
* General AI agent observability platforms — all now offering drift/anomaly detection as a core feature: Arize (AI Observability/Phoenix), AgentOps, Galileo, Langfuse, LangSmith, Latitude, Fiddler, Braintrust, MLflow, Helicone, Opik (Comet).
* One platform explicitly markets a **"Governed Agent Runtime"** with **"Decision Traces"** that capture reasoning paths, policy evaluations, and context quality specifically to detect **behavioral, model, and policy drift** — i.e., TaskAnchor's exact pitch, already shipping.
* Industry commentary explicitly notes that as of 2026, enterprises are moving from evaluating many tools to **picking 1–2 winners per category** — i.e., the consolidation phase has already begun for this category.

**Net effect:** "Goal drift detection" is no longer a differentiator. It is table-stakes marketing copy across a field of companies with $50M–$1B+ in funding and engineering teams an order of magnitude larger than 3 people.

## 5.4 AI Agent Red-Teaming & Benchmark Landscape

This is the sub-domain most relevant to ThreatBench, and it is now **saturated**:

* Academic benchmarks covering agentic attack surfaces: AgentDojo, InjecAgent, AgentHarm, ART, b³, ASB, Agent-SafetyBench, SHADE-Arena, OS-Harm, WASP, and DTap/DTap-Bench — all published within roughly the last two years, from well-resourced labs.
* Commercial/open-source tooling: PromptFoo (acquired by OpenAI), DeepEval, Microsoft's PyRIT, Mindgard, Confident AI, Lakera Gandalf.
* Frontier-lab practice has moved toward **private, non-public adversarial datasets** because public benchmarks suffer from "evaluation awareness" — models recognizing and gaming known public test sets (one study found a ~10x difference in models verbalizing evaluation-awareness on public vs. internal benchmarks).
* Industry-wide data: estimated **$2.3B** in losses from prompt-injection attacks (+340% YoY), **88%** of organizations running agents report confirmed or suspected security incidents, and current detection methods catch only **~23%** of sophisticated prompt-injection attempts. A January 2026 incident (the "OpenClaw" agent framework) shipped with 512 vulnerabilities including a one-click RCE, with 1,800+ exposed instances and 336 malicious marketplace plugins within a week.

**Net effect:** The remaining novel research frontier (benchmark contamination, evaluation awareness) is PhD-lab territory. A new benchmark from a 3-person team would have no realistic path to adoption or citation against this field, and zero monetization path.

## 5.5 Net Implications Summary

| Finalist | Original Assessment | Updated Assessment | Direction |
|---|---|---|---|
| PreFlight | Strong but #2 priority | **Genuine, narrow, regulator-backed gap** | ↑ |
| ThreatBench | Strong research, weak business | **Saturated; research edge gone too** | ↓↓ |
| TaskAnchor | #1 — highest ceiling | **Core moat commoditized; execution gap widened** | ↓↓↓ |

---

# 6. Why ThreatBench Was Rejected

**Verdict: Rejected (reconfirmed with stronger evidence).**

1. **No remaining gap.** The agentic-attack benchmark space — AgentDojo, InjecAgent, AgentHarm, ASB, Agent-SafetyBench, SHADE-Arena, OS-Harm, WASP, b³, DTap-Bench — already covers prompt injection, tool abuse, data exfiltration, and multi-step agent robustness across diverse environments. A new benchmark would need to articulate a genuinely novel angle that dozens of well-resourced academic teams have not already covered.
2. **The remaining frontier is too hard.** The one acknowledged open problem — benchmark contamination / evaluation awareness — is a measurement-science problem being tackled by frontier labs (Anthropic, Microsoft, UK AISI) with resources far beyond a 3-person team.
3. **No monetization path.** Even OpenAI chose to *acquire* an existing open-source evaluation tool (PromptFoo) rather than build bespoke benchmarking in-house — strong evidence that benchmarks are infrastructure components, not standalone businesses.
4. **Research novelty downgraded.** What was scored 10.0/10 in `06_Finalist_Comparison.md` is now realistically ~6.5/10 — the field moved from "wide open" to "crowded" in roughly 18 months.

**Disposition:** ThreatBench is retained only as an **internal validation asset** — its open corpora (AgentDojo, InjecAgent, ASB, etc.) become seed attack-payload libraries for PreFlight's dynamic simulation engine. It is not pursued as a product or as a primary research thrust.

---

# 7. Why TaskAnchor Was Rejected

**Verdict: Rejected (reversal of the Phase 2 decision).**

This was the hardest call, because TaskAnchor genuinely has the best raw market-size and willingness-to-pay story of the three. It is rejected **as currently scoped** for the following reasons:

1. **The core moat claim is now false.** The differentiation table in `09_Competitor_Analysis.md` asserted TaskAnchor uniquely provided "Goal Drift Detection." As of April 2026, goal drift / behavioral drift detection is a marketed, shipping feature across multiple platforms (Arize, AgentOps, Galileo, and at least one platform — "Governed Agent Runtime" with "Decision Traces" — built specifically around this exact framing). The single load-bearing row of TaskAnchor's competitive-moat table no longer holds.
2. **Competitive intensity moved from Medium to Severe.** $3.6B has flowed into this category in roughly the period since the original analysis, including direct competitors (WitnessAI, Noma Security, Pillar Security, AIM Security, Operant AI) and adjacent observability platforms that have absorbed the security-monitoring use case.
3. **The technical feasibility gap, already the worst of the three finalists (7.0/10), has widened.** Framework-agnostic, real-time instrumentation across LangChain, CrewAI, AutoGen, the OpenAI Agents SDK, and MCP — at the breadth claimed by competitors (one comparison lists 900+ integrations for a single platform) — is a multi-year, multi-team distributed-systems effort. It was already the highest-risk item in `10_Go_NoGo_Decision.md` ("Technical Complexity: High"); that risk has not been mitigated, and the bar it must clear has risen.
4. **The dataset problem remains unsolved.** TaskAnchor requires real agent execution traces with labeled goal-drift ground truth. No such labeled public dataset exists, and "drift" is inherently context-dependent — this was already the team's stated weakness and nothing in the intervening period has changed it.
5. **Large TAM = crowded TAM, at this stage, for this team.** The Sequoia Partner's framing from the panel review applies directly: TaskAnchor's market-size slide is the best of the three, but that size is precisely *why* it is now a land-grab among players with 10–50x the team's capital and headcount. Large, obviously-hot markets attract capital fastest — an unfunded student team entering 12–18 months after the funding wave began is structurally disadvantaged.

**Disposition:** TaskAnchor's vision is **not discarded** — it is demoted to a **Phase 2/Phase 3 roadmap item** for the winning project (see Section 8 and `12_Product_Requirements_Document.md`, Section 15). A narrow slice of TaskAnchor's value — *continuous permission-drift detection* — becomes a natural extension of PreFlight once the core scanner is built, without requiring the team to compete on runtime-observability breadth from day one.

---

# 8. Why PreFlight Won

**Verdict: Selected, repositioned as "MCP & Agent Permission Attack-Surface Scanner."**

PreFlight wins not because it has the largest TAM — it doesn't — but because it is the only finalist that survives simultaneous stress-testing on feasibility, differentiation, dataset availability, *and* market timing:

1. **Real, regulator-backed customer pain.** The NIST AI Agent Standards Initiative, the AIUC-1 Q2 2026 update (mandatory third-party access monitoring, MCP tool-call governance), and EU AI Act Article 55's adversarial-testing requirement are all actively *creating* demand for pre-deployment, permission-aware agent risk assessment — and none of them yet point to a dominant incumbent product in this specific niche.
2. **A genuine, narrow, defensible gap.** The competitive landscape splits cleanly into two camps: (a) authorization/identity plumbing for MCP (Aembit, TrueFoundry) and (b) generic prompt-injection red-teaming/benchmarking (Mindgard, PyRIT, Confident AI). **Nobody combines a permission-graph attack-path model with LLM-driven dynamic simulation of what an agent would actually do, given a specific tool grant, under adversarial input.** That combination — "BloodHound for AI agents, with a built-in attacker" — is PreFlight's wedge.
3. **Feasible for this exact team, on this exact hardware.** The core engine is graph construction, graph traversal/attack-path search, and orchestrated LLM API calls against a sandbox — not large-scale model training. The RTX 2050/24GB constraint that makes TaskAnchor's instrumentation breadth unrealistic is a non-issue for PreFlight.
4. **Dataset-rich without production telemetry.** Thousands of open-source MCP server manifests are publicly available, MCP-specific CVEs are documented (CVSS 7.3–9.6, 437,000+ affected installations), and the academic attack corpora rejected as ThreatBench's *product* (AgentDojo, InjecAgent, ASB, Agent-SafetyBench) become PreFlight's *seed attack library* — turning a competitor's saturation into PreFlight's free R&D head start.
5. **A credible land-and-expand story.** Point-in-time/CI-CD scanning (the Snyk playbook: static analysis → CI gate → continuous monitoring) gives PreFlight a controlled, incremental path toward exactly the kind of "continuous drift detection" value TaskAnchor wanted to deliver on day one — but earned, sequenced, and scoped to what 3 people can actually ship.

---

# 9. Final Market Assessment

## Market Category

PreFlight sits inside the **MCP/agentic AI security** category — the specific sub-category called out at RSAC 2026 as "the new category," with ~$40M in disclosed dedicated funding to date (still early relative to the $3.6B flowing into broader agentic AI security).

## TAM / SAM / SOM

* **TAM:** Enterprise AI agent governance & security spend, multi-billion-dollar over the next decade (consistent with `08_Market_Validation.md`'s broader category estimate).
* **SAM:** Organizations deploying MCP-connected agents and AI platform teams subject to emerging standards (NIST AI Agent Standards Initiative, AIUC-1, EU AI Act Article 55) — a rapidly growing population given "tens of thousands" of MCP servers already deployed by 2025.
* **SOM (initial):** AI-native startups, SaaS companies, and MSSPs building or operating MCP-connected agents who need a pre-deployment / CI-integrated permission risk assessment to satisfy internal security review or external compliance questionnaires.

## Customer Personas

* **AI Platform Engineer** — wants a CI/CD gate that catches over-permissioned agent configs before merge.
* **AppSec / Security Engineer** — wants a BloodHound-style attack-path view of "what could go wrong" across the agent fleet's tool grants.
* **Compliance Officer** — wants exportable evidence mapping scan results to NIST/AIUC-1/OWASP/EU AI Act control IDs.
* **MSSP / Consultancy** — wants a repeatable assessment product to sell as a service to multiple clients (channel opportunity).

## Customer Willingness to Pay

Moderate-to-high and rising. Security teams already budget for SAST/DAST/CSPM-class tooling; compliance mandates convert "nice to have" into "must have" line items. The risk (also true of the original ThreatBench critique) is recurring-revenue durability for a point-in-time scan — mitigated by the CI/CD and continuous-drift roadmap (Section 15 of `12_Product_Requirements_Document.md`).

---

# 10. Final Technical Assessment

## Updated Comparative Scoring

| Metric | PreFlight (Updated) | ThreatBench (Updated) | TaskAnchor (Updated) |
|---|---|---|---|
| Market Opportunity | 9.0 | 6.0 | 9.0 |
| Startup Potential | 8.0 | 3.0 | 4.0 |
| Research Potential | 9.0 | 6.5 | 7.0 |
| Recruiter Value | 9.0 | 7.5 | 9.5 |
| **Technical Feasibility** | **8.5** | 8.0 | **5.5** |
| SIH Potential | 9.0 | 6.5 | 8.0 |

## Key Technical Feasibility Drivers (PreFlight)

* **Permission Graph Builder** — parsing MCP manifests / tool definitions into a typed graph (NetworkX/Neo4j) is standard graph-engineering work, well within the team's Python skill set.
* **Static Attack Path Analysis** — graph traversal/search algorithms (shortest path to "sensitive sink" nodes) are computationally cheap; no GPU dependency.
* **Dynamic Simulation Engine** — orchestrates LLM API calls (attacker, victim-agent simulator, judge) against containerized mock tools; GPU is optional (only needed if running local open-weight models for cost control during development — RTX 2050/24GB is sufficient for quantized 7–13B models via Ollama).
* **No production telemetry dependency** — unlike TaskAnchor, PreFlight does not require live customer agent traffic to function or to validate, removing the single largest feasibility blocker identified in the original analysis.

## Architecture Complexity Comparison

| Component | PreFlight | TaskAnchor |
|---|---|---|
| Core data structure | Permission graph (static, per-scan) | Live execution traces (streaming, per-agent) |
| Primary infra challenge | Sandbox fidelity for mock tools | Cross-framework instrumentation breadth |
| ML role | LLM-as-attacker/judge, embeddings for CVE matching | Behavioral anomaly models requiring labeled drift data |
| Deployment model | On-demand scan / CI job | Always-on production agent |

---

# 11. Final Startup Assessment

## Positioning

"PreFlight" → **MCP & Agent Permission Attack-Surface Scanner** — a pre-deployment and CI/CD-integrated tool that builds a permission graph from an organization's MCP servers and agent tool grants, statically identifies high-risk attack paths, and dynamically validates the most dangerous paths via sandboxed LLM-driven exploit simulation.

## Moat / Defensibility

* **Technical moat:** the combined permission-graph + dynamic-exploit-simulation engine — a genuinely uncommon pairing in the current vendor landscape.
* **Data moat (emergent):** over time, an anonymized corpus of real-world MCP permission configurations and successful/unsuccessful simulated exploits becomes a proprietary dataset competitors lack — directly analogous to the "data moat" originally claimed (but unrealized) for TaskAnchor, achieved here through a much lower-friction collection mechanism (a scan, not continuous production access).
* **Standards moat:** early, deep alignment to NIST AI Agent Standards Initiative / AIUC-1 / OWASP Agentic / MITRE ATLAS gives PreFlight a procurement narrative that's harder for generic red-teaming tools to replicate quickly.

## Monetization Path

1. **Free/open-core CLI scanner** (adoption + community + dataset flywheel).
2. **CI/CD-integrated SaaS** (GitHub Action / GitLab CI gate) — Snyk-style land.
3. **Continuous monitoring add-on** — scheduled re-scans, permission-drift alerts (controlled, scoped slice of the original TaskAnchor vision).
4. **MSSP/consultancy tier** — multi-tenant assessments-as-a-service.

## Risks

* **Recurring revenue durability** for point-in-time scans — mitigated by CI/CD + drift-alert tiers.
* **Sandbox fidelity gap** — mock tools may not perfectly represent real backends; mitigated by using real open-source MCP server schemas as fixtures.
* **Category crowding over time** — the broader AI security market is hot; mitigated by the narrow initial positioning and standards-alignment differentiation.

## Updated Probability Estimates

* **Startup success (independent company):** 20–30%
* **Sustainable niche SaaS:** 30–40%
* **Acqui-hire / technology acquisition** (by an MCP-security, CSPM/CIEM, or AppSec platform vendor): 30–40% — and notably, the M&A environment ($96B in 2025 cybersecurity M&A) makes this a *credible* and *attractive* outcome, not a fallback.

---

# 12. Final Research Assessment

## Research Questions

1. **Permission-graph-aware automated red-teaming** — given an agent's exact tool/scope grants (modeled as a graph), can an LLM-driven attacker agent more efficiently discover viable exploit chains than scenario-agnostic fuzzing (as used by AgentDojo/ASB-style benchmarks)?
2. **Attack path scoring** — can a risk score combining graph topology (hop count, privilege level, data sensitivity of reachable nodes) with empirical simulation success rate predict real-world exploitability better than either signal alone?
3. **Transferability** — do attack paths discovered against mock/sandboxed tool implementations transfer to real implementations of the same MCP tool schema?
4. **Judge reliability** — how reliable is an LLM-as-judge for determining whether a simulated exploit "succeeded" (e.g., sensitive data exfiltrated), and how does this compare to rule-based verification where possible?

## Novelty Position

This extends the AgentDojo/InjecAgent/ASB/Agent-SafetyBench lineage (attack taxonomies and benchmarks for agent tool-use) into a **permission-graph-conditioned** setting — i.e., not "is this agent vulnerable to attack X in general" but "given *this specific* agent's *actual* tool grants, what is the realistic blast radius." This conditioning on a structured permission model is the genuinely under-explored angle identified by the panel.

## Target Venues

* Security venues: USENIX Security, NDSS, ACSAC (workshop tracks on AI/agent security).
* AI safety venues: NeurIPS/ICLR workshops on agent safety and red-teaming.
* Industry: OWASP Agentic AI Security project contributions; MITRE ATLAS case study submissions.

---

# 13. Smart India Hackathon Assessment

## Theme Alignment

* Cybersecurity
* AI Governance / Responsible AI
* Digital Infrastructure Security
* Emerging Technologies (Agentic AI / MCP)

## Problem Statement Framing

"As Indian enterprises and government digital infrastructure adopt MCP-connected AI agents at scale, there is no systematic way to assess, before deployment, what an agent could be tricked into doing with the access it has been granted. PreFlight provides an automated permission-graph attack-surface scanner that identifies and simulates realistic exploit chains before they reach production."

## Demo Narrative

A live, high-impact demo: upload an agent's MCP configuration (e.g., an agent with CRM-read + email-send tools), watch PreFlight build the permission graph, highlight the data-exfiltration attack path, and then **show the simulated attack succeeding** (a sandboxed attacker LLM crafts a prompt-injection payload that causes the victim agent to email customer PII to an external address) — followed by the remediation recommendation (scope the email tool, add human-in-the-loop).

## SIH Score Rationale

Updated to **9.0/10** (from 7.5/10 originally) — the dynamic simulation demo is significantly more visceral and judge-friendly than either a dashboard (TaskAnchor) or a leaderboard (ThreatBench), and the national-relevance narrative (India's rapidly growing MCP/agent ecosystem and digital infrastructure exposure) is strong.

---

# 14. Go / No-Go Decision

## Final Evaluation Scores — PreFlight (Repositioned)

| Category | Score |
|---|---|
| Market Opportunity | 9/10 |
| Technical Depth | 8.5/10 |
| Recruiter Value | 9/10 |
| Research Potential | 9/10 |
| Startup Potential | 8/10 |
| Feasibility | 8.5/10 |
| Innovation | 8.5/10 |

### Overall Score: **8.6 / 10**

## Decision

## GO

PreFlight (repositioned as the MCP & Agent Permission Attack-Surface Scanner) is approved to proceed to Product Definition and Architecture Design, **conditional on the following constraints**, which directly address the failure modes identified during re-evaluation:

1. **Scope discipline.** The MVP is the permission-graph + static attack-path engine + sandboxed dynamic simulation engine + reporting dashboard. Continuous/production runtime monitoring (the rejected TaskAnchor scope) is explicitly Phase 2+ roadmap, not MVP.
2. **Standards-first framing.** Every finding must be mappable to a specific NIST AI Agent Standards Initiative / AIUC-1 / OWASP Agentic / MITRE ATLAS reference from the outset — this is the procurement and SIH narrative.
3. **Leverage rejected assets.** ThreatBench's surveyed corpora (AgentDojo, InjecAgent, ASB, Agent-SafetyBench) are adopted as the seed attack-payload library for the dynamic simulation engine — converting a "rejected idea" into a research and engineering accelerant.
4. **Sandbox-only execution.** The dynamic simulation engine must never execute against real production systems or credentials — sandboxed mock tools only, by design (see `12_Product_Requirements_Document.md`, Section 18).

---

# 15. Final Recommendation

Proceed immediately to:

* `12_Product_Requirements_Document.md` — Product Requirements Document for PreFlight (MCP & Agent Permission Attack-Surface Scanner)
* `13_System_Architecture.md`
* `14_AI_Architecture.md`
* `15_Tech_Stack.md`
* `16_Development_Roadmap_15_Months.md`

## Status

**APPROVED FOR DEVELOPMENT — PreFlight (MCP & Agent Permission Attack-Surface Scanner)**

This document constitutes the final, official record of the AI Startup Discovery Lab's project selection process. All prior selection records (`07_Selected_Startup.md`, `10_Go_NoGo_Decision.md`) remain part of the historical research trail but are no longer the operative decision.
