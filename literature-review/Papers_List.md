# Papers_List.md — PreFlight Literature Review
**Project:** PreFlight — AI Agent Permission Attack Surface Scanner  
**Phase:** Literature Review  
**Last Updated:** 2026-06-19  
**Total Papers:** 36  

---

## Tier Classification Criteria

| Tier | Label | Definition |
|------|-------|-----------|
| **Tier 1** | Core | Directly addresses the PreFlight problem space (MCP security, agent permission models, attack surface analysis, prompt injection in agentic systems). Indispensable for the research paper and design. |
| **Tier 2** | Important | Closely related foundational or benchmarking work; substantially informs threat models, evaluation methods, or safety arguments. |
| **Tier 3** | Background | Broader context — general AI safety, multi-agent systems, or adjacent security domains. Useful for related-work sections and framing. |

---

## Category Codes

| Code | Area |
|------|------|
| **AS** | Agent Security |
| **PI** | Prompt Injection |
| **MCP** | MCP Security |
| **TP** | Tool Permissions |
| **AG** | Attack Graphs |
| **AIS** | AI Safety |
| **MAS** | Multi-Agent Security |

---

## Papers List

| Paper_ID | Title | Authors | Year | Category | Research Area | URL / DOI | Tier |
|----------|-------|---------|------|----------|--------------|-----------|------|
| P01 | LLM and AI Agents for Autonomous Systems: Security, Safety, and Applications | M. A. Ferrag, A. Lakas, N. Tihanyi, M. Debbah | 2026 | AS, MAS | LLM agents in autonomous systems, security threats, red-teaming | IEEE Access Vol.7, 2026 | Tier 3 |
| P02 | AI Agents Under Threat: A Survey of Key Security Challenges and Future Pathways | Z. Deng, Y. Guo, C. Han, W. Ma, J. Xiong, S. Wen, Y. Xiang | 2025 | AS | AI agent security survey, threat taxonomy, knowledge gaps | https://doi.org/10.1145/3716628 — ACM Comput. Surv. 57(7):182, Feb 2025 | Tier 1 |
| P03 | Multi-Agent Security: Taxonomy, Threat Landscape, and Future Research Directions | C. S. de Witt et al. (Oxford/Multi-institution) | 2026 | MAS, MCP, AIS | Multi-agent security, covert collusion, steganography, MCP governance | arXiv preprint v1.0, 2026 | Tier 1 |
| P04 | Not What You've Signed Up For: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection | K. Greshake, S. Abdelnabi, S. Mishra, C. Endres, T. Holz, M. Fritz | 2023 | PI, AS | Indirect prompt injection, real-world LLM applications | https://doi.org/10.1145/3605764.3623985 — AISec @ CCS 2023 | Tier 1 |
| P05 | Identifying the Risks of LM Agents with an LM-Emulated Sandbox | Y. Ruan, H. Dong, A. Wang, S. Pitis, Y. Zhou, J. Ba, Y. Dubois, C. Maddison, T. Hashimoto | 2024 | AS, TP | LM agent risk, sandboxed emulation, tool misuse | ICLR 2024 | Tier 1 |
| P06 | AgentDojo: A Dynamic Environment to Evaluate Prompt Injection Attacks and Defenses for LLM Agents | E. Debenedetti, J. Zhang, M. Balunovic, L. Beurer-Kellner, M. Fischer, F. Tramèr | 2024 | PI, AS | Prompt injection benchmark, 97 tasks, 629 security test cases, tool-calling agents | https://doi.org/10.52202/079017-2636 — NeurIPS 2024 | Tier 1 |
| P07 | InjecAgent: Benchmarking Indirect Prompt Injections in Tool-Integrated LLM Agents | Q. Zhan, Z. Liang, Z. Ying, D. Kang | 2024 | PI, AS | Indirect prompt injection benchmark, tool-integrated agents | arXiv:2403.02691 | Tier 1 |
| P08 | Formalizing and Benchmarking Prompt Injection Attacks and Defenses | Y. Liu, Y. Jia, R. Geng, J. Jia, N. Z. Gong | 2023 | PI | Formal model for prompt injection, benchmark | arXiv:2310.12815 | Tier 2 |
| P09 | Prompt Injection Attack Against LLM-Integrated Applications | Y. Liu, G. Deng, Y. Li, K. Wang, T. Zhang, Y. Liu, H. Wang, Y. Zheng, Y. Liu | 2023 | PI, AS | Prompt injection taxonomy, attack vectors against LLM apps | arXiv:2306.05499 | Tier 2 |
| P10 | The Instruction Hierarchy: Training LLMs to Prioritize Privileged Instructions | E. Wallace, K. Xiao, R. Leike, L. Weng, J. Heidecke, A. Beutel | 2024 | PI, AIS | Instruction hierarchy, privilege levels, injection defense | arXiv:2404.13208 | Tier 2 |
| P11 | Tensor Trust: Interpretable Prompt Injection Attacks from an Online Game | S. Toyer, O. Watkins, E. Mendes, J. Svegliato, L. Bailey, T. Wang, I. Ong, K. Elmaaroufi, P. Abbeel, T. Darrell, A. Ritter, S. Russell | 2023 | PI | Prompt injection game, interpretable attack patterns | arXiv:2311.01011 | Tier 2 |
| P12 | Ignore This Title and HackAPrompt: Exposing Systemic Vulnerabilities of LLMs Through a Global Prompt Hacking Competition | S. Schulhoff, J. Pinto, A. Khan, L. Bouchard, C. Si, J. Boyd-Graber et al. | 2023 | PI | Prompt hacking competition, systemic LLM vulnerabilities | EMNLP 2023 | Tier 2 |
| P13 | Benchmarking and Defending Against Indirect Prompt Injection Attacks on Large Language Models | J. Yi, Y. Xie, B. Zhu, E. Kiciman, G. Sun, X. Xie, F. Wu | 2023 | PI, AS | Indirect prompt injection benchmark and defense | arXiv:2312.14197 | Tier 2 |
| P14 | SecGPT: An Execution Isolation Architecture for LLM-Based Systems | Y. Wu, F. Roesner, T. Kohno, N. Zhang, U. Iqbal | 2024 | AS, TP | Execution isolation, LLM-based systems, sandboxing | arXiv:2403.04960 | Tier 1 |
| P15 | StruQ: Defending Against Prompt Injection with Structured Queries | S. Chen, J. Piet, C. Sitawarin, D. Wagner | 2024 | PI | Structured query defense, prompt injection | arXiv:2402.06363 | Tier 2 |
| P16 | Defending Against Indirect Prompt Injection Attacks With Spotlighting | K. Hines, G. Lopez, M. Hall, F. Zarfati, Y. Zunger, E. Kiciman | 2024 | PI | Spotlighting defense, indirect prompt injection | arXiv:2403.14720 | Tier 2 |
| P17 | Neural Exec: Learning (and Learning from) Execution Triggers for Prompt Injection Attacks | D. Pasquini, M. Strohmeier, C. Troncoso | 2024 | PI | Learned execution triggers, prompt injection | arXiv:2403.03792 | Tier 2 |
| P18 | ToolEmu: Tool Emulation for Evaluating AI Agent Safety | S. Ruan et al. (Stanford) | 2023 | AS, TP | Tool emulation, agent safety evaluation, LLM agents | arXiv (ToolEmu) 2023 | Tier 1 |
| P19 | Coercing LLMs to Do and Reveal Almost Anything | J. Geiping, A. Stein, M. Shu, K. Saifullah, Y. Wen, T. Goldstein | 2024 | PI, AIS | LLM coercion, adversarial attacks, jailbreaking | arXiv:2402.14020 | Tier 2 |
| P20 | Can LLMs Separate Instructions From Data? And What Do We Even Mean By That? | E. Zverev, S. Abdelnabi, M. Fritz, C. H. Lampert | 2024 | PI, AIS | Instruction-data separation, LLM robustness | arXiv:2403.06833 | Tier 2 |
| P21 | Model Context Protocol (MCP): Landscape, Security Threats, and Future Research Directions | X. Hou, Y. Zhao, S. Wang, H. Wang | 2025 | MCP, AS | MCP architecture, security threat landscape, attack vectors | arXiv:2503.23278 | Tier 1 |
| P22 | Securing Agentic AI: A Comprehensive Threat Model and Mitigation Framework for Generative AI Agents | V. S. Narajala, O. Narayan | 2025 | AS, MCP, TP | Agentic AI threat model, MCP security, mitigation framework | arXiv 2025 | Tier 1 |
| P23 | MPMA: Preference Manipulation Attack Against Model Context Protocol | Z. Wang, R. Zhang, Y. Liu, W. Fan, W. Jiang, Q. Zhao, H. Li, G. Xu | 2026 | MCP, PI | MCP tool selection manipulation, DPMA, GAPMA genetic-based attacks | AAAI-26; arXiv (Wang et al. 2025) | Tier 1 |
| P24 | Permission Manifests for Web Agents | S. Marro, A. Chan, X. Ren, L. Hammond, J. Wright, G. Wanga, T. Piccardi, N. Campos, T. South, J. Yu, S. Sengupta, E. Sommerlade, A. Pentland, P. Torr, J. Pei | 2026 | TP, AS | Permission manifests, web agent authorization, least-privilege | arXiv:2601.02371 | Tier 1 |
| P25 | ChainReactor: Automated Privilege Escalation Chain Discovery via AI Planning | G. De Pasquale, I. Grishchenko, R. Iesari, G. Pizarro, L. Cavallaro, C. Kruegel, G. Vigna | 2024 | AG, AS | Automated privilege escalation, AI planning, attack chains | USENIX Security 2024 | Tier 1 |
| P26 | From Prompt Injections to Protocol Exploits: Threats in LLM-Powered AI Agents Workflows | M. A. Ferrag, N. Tihanyi, D. Hamouda, L. Maglaras, A. Lakas, M. Debbah | 2025 | PI, AS, MCP | Protocol exploits, LLM agent workflows, MCP attack surfaces | arXiv:2506.23260 | Tier 1 |
| P27 | Taxonomy of Failure Modes in Agentic AI Systems | P. Bryan, G. Severi, J. de Gruyter, D. Jones et al. (Microsoft AI Red Team) | 2025 | AS, MAS | Failure mode taxonomy, agentic AI, red-teaming | Microsoft AI Red Team Whitepaper, April 2025 | Tier 2 |
| P28 | Secret Collusion Among AI Agents: Multi-Agent Deception via Steganography | S. R. Motwani, M. Baranchuk, M. Strohmeier, V. Bolina, P. H. S. Torr, L. Hammond, C. S. de Witt | 2024 | MAS, AIS | Steganographic covert channels, multi-agent deception, NeurIPS 2024 | NeurIPS 2024; arXiv:2402.07510 | Tier 2 |
| P29 | Institutional AI: Governing LLM Collusion in Multi-Agent Cournot Markets via Public Governance Graphs | M. B. Syrnikov, F. Pierucci, M. Galisai, M. Prandi, P. Bisconti, F. Giarrusso, O. Sorokoletova, V. Suriani, D. Nardi | 2026 | MAS, AIS | LLM collusion governance, governance graphs, runtime enforcement | arXiv:2601.11369 | Tier 3 |
| P30 | OMNI-LEAK: Orchestrator Multi-Agent Network Induced Data Leakage | A. Naik, J. Culligan, Y. Gal, P. Torr, R. Aljundi, A. Paren, A. Bibi | 2026 | MAS, AS | Multi-agent data leakage, orchestrator vulnerabilities | arXiv:2602.13477 | Tier 2 |
| P31 | Authenticated Delegation and Authorized AI Agents | T. South, S. Marro, T. Hardjono, R. Mahari, C. D. Whitney, D. Greenwood, A. Chan, A. Pentland | 2025 | TP, AS | Agent authentication, delegation, authorization | arXiv:2501.09674 | Tier 2 |
| P32 | Policy Compiler for Secure Agentic Systems | N. Palumbo, S. Choudhary, J. Choi, P. Chalasani, S. Jha | 2026 | TP, AS | Policy compilation, secure agentic systems, access control | arXiv Feb 2026 | Tier 1 |
| P33 | Agent Safety Bench / Agent-SafetyBench: Evaluating the Safety of LLM Agents | (Various; referenced in PreFlight AI Architecture doc) | 2024 | AIS, AS | LLM agent safety evaluation benchmark | Referenced as ASB corpus in PreFlight docs | Tier 2 |
| P34 | A Scalable Communication Protocol for Networks of Large Language Models | S. Marro, E. La Malfa, J. Wright, G. Li, N. Shadbolt, M. Wooldridge, P. Torr | 2024 | MAS | Scalable communication, LLM networks, agent protocols | arXiv:2410.11905 | Tier 3 |
| P35 | HarmBench: A Standardized Evaluation Framework for Automated Red Teaming and Robust Refusal | M. Mazeika, L. Phan, X. Yin, A. Zou, Z. Wang, N. Mu, E. Sakhaee, N. Li, S. Basart, B. Li, D. Forsyth, D. Hendrycks | 2024 | AIS, PI | Red-teaming benchmark, refusal evaluation, jailbreaking | arXiv:2402.04249 | Tier 2 |
| P36 | Foundational Challenges in Assuring Alignment and Safety of Large Language Models | U. Anwar, A. Saparov, J. Rando, D. Paleka, M. Turpin, P. Hase, E. S. Lubana, E. Jenner, S. Casper, O. Sourbut, B. Edelman, Z. Zhang, M. Günther, A. Korinek, J. Hernandez-Orallo, L. Hammond et al. | 2024 | AIS | LLM alignment, safety challenges, threat survey | TMLR May 2024; arXiv:2404.09932 | Tier 3 |

---

## Tier Summary

| Tier | Count | Papers |
|------|-------|--------|
| **Tier 1 — Core** | 14 | P02, P03, P04, P05, P06, P07, P14, P18, P21, P22, P23, P24, P25, P26, P32 |
| **Tier 2 — Important** | 16 | P08, P09, P10, P11, P12, P13, P15, P16, P17, P19, P20, P27, P28, P30, P31, P33, P35 |
| **Tier 3 — Background** | 6 | P01, P29, P34, P36 |

---

## Category Distribution

| Category | Papers |
|----------|--------|
| **AS — Agent Security** | P01, P02, P03, P04, P05, P06, P07, P09, P14, P18, P21, P22, P23, P24, P26, P27, P30, P31, P32 |
| **PI — Prompt Injection** | P04, P06, P07, P08, P09, P10, P11, P12, P13, P15, P16, P17, P19, P20, P23, P26 |
| **MCP — MCP Security** | P03, P21, P22, P23, P26 |
| **TP — Tool Permissions** | P05, P14, P18, P24, P31, P32 |
| **AG — Attack Graphs** | P25 |
| **AIS — AI Safety** | P03, P10, P19, P20, P28, P29, P33, P35, P36 |
| **MAS — Multi-Agent Security** | P01, P03, P27, P28, P29, P30, P34 |

---

## Ranking Rationale Notes

**Tier 1 (Core):** These papers either (a) directly attack or defend MCP servers (P21, P22, P23, P26, P32), (b) provide the primary benchmarks PreFlight's evaluation will reference (P06 AgentDojo, P07 InjecAgent, P18 ToolEmu), (c) demonstrate tool permission exploits and sandbox escape (P05, P14, P25), (d) formalize the multi-agent threat landscape most directly relevant to PreFlight's scope (P02, P03), or (e) address agent-side access control primitives (P24).

**Tier 2 (Important):** These papers are foundational to understanding prompt injection mechanics, evaluation methodology, instruction hierarchy defenses, and multi-agent deception — all informing PreFlight's threat model and defense design, but not narrowly scoped to MCP or permission graphs.

**Tier 3 (Background):** Broad surveys of AI safety, multi-agent protocol design, and LLM alignment that provide conceptual scaffolding for the introduction and related-work sections without directly mapping to PreFlight's attack surface.

---

*Document generated for the PreFlight Literature Review phase. To be cross-referenced with Literature_Matrix.xlsx and Research_Gap.md.*
