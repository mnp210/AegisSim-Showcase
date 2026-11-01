# AegisSim: Research Contributions & Literature Positioning

## 1. Primary Research Contributions

AegisSim contributes an open-source, mathematically reproducible framework for evaluating and constraining the security behavior of autonomous AI agents in enterprise computing environments. Specifically, the framework introduces:

1. **Deterministic Execution-Layer Reference Monitor (PEP)**: An instantiation of James Anderson's classical reference monitor concept (Anderson, 1972) specifically engineered for autonomous agent execution loops. It decouples access control and security policy enforcement from language model reasoning, executing 10 sequential procedural inspection stages prior to runtime tool dispatch.
2. **Multi-Tenant Relational Enterprise Sandbox**: A fully emulated 21-table enterprise schema in PostgreSQL featuring organizational partitioning, a 4-tier hierarchical sensitivity classification (`PUBLIC`, `INTERNAL`, `CONFIDENTIAL`, `RESTRICTED`), realistic relational entities (departments, users, files, approval queues), and unindexed decoy canary honeytokens.
3. **Synchronous Human-in-the-Loop Review Mechanics**: A non-blocking, synchronous approval gateway that intercepts actions exceeding explicit risk thresholds ($R \ge 80$), preserving agent state snapshots in a dedicated queue without deadlocking server event loops.
4. **Tamper-Evident Event Ledger**: Sequential SHA-256 block hash chaining linking every tool invocation, observation, and policy verdict, delivering verifiable forensic audit trails.
5. **Controlled Pre-Registered Benchmark Suite**: A rigorous 241-trial experimental evaluation spanning five formal research questions (RQ1–RQ5), featuring pre-registered ground truth, frozen seeds, logical tenant isolation, and transparent reporting of both successes and failures.

---

## 2. Comparison with Conventional Benchmarks and Guardrail Approaches

| Architectural Dimension | Neural Evaluators / "LLM-as-a-Judge" (Zheng et al., 2023) | Guardrail Toolkits (NeMo Guardrails, Rebedea et al., 2023; Llama Guard) | Offline Attack Benchmarks (InjecAgent, Zhan et al., 2024; ToolEmu, Ruan et al., 2023) | AegisSim (`v0.1.0-alpha`) |
| :--- | :--- | :--- | :--- | :--- |
| **Enforcement Layer** | Secondary LLM prompt evaluation | Conversational dialogue rails (Colang) evaluated via embeddings or auxiliary models | Static prompt datasets or LM-emulated tool stubs | **Deterministic reference monitor at runtime tool boundary** |
| **Decision Determinism** | **Non-deterministic** ($\sigma^2 > 0$ across identical inputs) | Mixed; procedural rails deterministic, but embedding classifiers exhibit boundary drift | Deterministic offline string matching or simulated scoring | **100% deterministic decision logic** ($\sigma^2 = 0$ on defined rules) |
| **Susceptibility to Injection** | **Vulnerable**: secondary evaluating model can be influenced or jailbroken | Conversational rails vulnerable to out-of-distribution semantic evasion | N/A (Passive evaluation measuring base model vulnerability) | Policy engine executes zero LLM inference; syntactic heuristics remain susceptible to lexical evasion (`SC-04`, `SC-11`) |
| **Runtime Overhead** | Secondary model inference latency per evaluation step plus ongoing API token cost | Varies by rail configuration (embedding similarity, vector scoring, or auxiliary LLM call) | N/A (Offline batch evaluation; zero operational runtime overhead) | Procedural execution; empirical mean mediation latency of **$4.37\text{ ms}$** ($P_{95} = 5.98\text{ ms}$) measured on $N=120$ physical trials |
| **Operational Execution Environment** | Mocked function calls or text outputs | Conversational dialogue streams and application integrations | Static benchmark suites or LM-emulated tool environments | **Interactive 21-table PostgreSQL relational sandbox** |
| **Forensic Traceability** | Standard ephemeral application logs | Application telemetry and dialogue traces | Static benchmark run logs | **Sequential SHA-256 block hash ledger with chain verification** |
| **Failure Transparency** | Aggregate benchmark scores without root-cause failure cases | Operational exception logs and boundary classification metrics | Aggregate attack success rates across static test suites | **Pre-registered ground truth with deep failure case studies (`SC-04`, `SC-11`)** |

---

## 3. Positioning in Academic Literature

### Versus NeMo Guardrails (Rebedea et al., 2023)
NeMo Guardrails utilizes programmable conversational rails (Colang) evaluated by an LLM backend or semantic similarity models. While versatile for dialogue bounding, conversational rails are subject to semantic paraphrasing evasions and introduce inference overhead when calling auxiliary models. AegisSim mediates directly at the system and database boundary using exact schema validation, SQL AST parsers, relational tenant boundaries, and cryptographic ledger chaining.

### Versus InjecAgent (Zhan et al., 2024)
InjecAgent provides an extensive offline evaluation benchmark (1,054 test cases) measuring agent vulnerability to indirect prompt injections. However, it measures model vulnerability in isolation without runtime mediation. AegisSim evaluates an active execution-layer runtime containment platform that intercepts and blocks unauthorized tool actions in real time.

### Versus AgentBench (Liu et al., 2024)
AgentBench evaluates multi-turn task completion across diverse environments (OS, web, database), but focuses primarily on problem-solving competence rather than defensive access control boundaries, clearance hierarchies, or forensic non-repudiation.

### Versus ToolEmu (Ruan et al., 2023)
ToolEmu deploys an "LM-emulated sandbox" where a secondary language model emulates tool behavior and evaluates safety outcomes. AegisSim executes against a real relational database fabric and evaluates safety through deterministic reference monitor invariants, eliminating probabilistic hallucination from safety verdicts.

---

## 4. Why Deterministic Policy Enforcement Matters

In high-assurance enterprise computing (financial transactions, healthcare records, privileged infrastructure), access control is a mathematical mapping governed by **Saltzer and Schroeder's security design principles (1975)**:

1. **Economy of Mechanism**: Access decisions must be simple, auditable, and mathematically verifiable.
2. **Complete Mediation**: Every access attempt must be intercepted without exception.
3. **Fail-Safe Defaults**: Access decisions must default to denial unless an explicit authorization grant exists.

Entrusting authorization to probabilistic language models violates all three principles:
- An LLM cannot provide formal mathematical guarantees of non-bypassability.
- Neural judges suffer from prompt injection, hallucination, and semantic drift.
- A deterministic monitor running compiled regular expressions, AST parsers, and relational permission checks operates in an empirical mean of **$4.37\text{ ms}$**, executes invariant decision logic across identical inputs ($\sigma^2 = 0$ on defined rules), and does not rely on linguistic reasoning.

---

## 5. What the Experiments Demonstrate vs. What Remains Open

### Proven by Empirical Benchmark:
- Deterministic reference monitors provide zero-variance, sub-5ms policy enforcement across diverse tool-calling categories under defined rules.
- Pre-execution SQL AST parsing reliably blocks destructive database mutations (`DROP`, `TRUNCATE`, unconstrained `DELETE`).
- Synchronous HITL gating intercepts high-risk operations ($R \ge 80$) without causing event-loop deadlocks.
- Sequential SHA-256 block hashing reliably flags intermediate row modifications and deletions.

### What Remains Unproven / Open Research Challenges:
- **Semantic Evasion of Syntactic Heuristics**: As shown in `SC-04` and `SC-11`, when an agent is authorized for a tool, purely syntactic regex matching cannot detect all adversarial intents.
- **Hardware-Level Non-Repudiation**: The current ledger detects tampering at the application level, but does not independently prevent database superusers from recalculating hashes without external anchoring.
- **Scalability to Arbitrary Code Execution**: The current sandbox is relational and API-centric; sandboxing arbitrary untrusted bash scripts requires hypervisor-level isolation (e.g., Firecracker micro-VMs).
