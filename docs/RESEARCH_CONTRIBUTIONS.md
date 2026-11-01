# AegisSim: Research Contributions & Literature Positioning

## 1. Primary Research Contributions

AegisSim contributes an open-source, mathematically reproducible framework for evaluating and constraining the security behavior of autonomous AI agents in enterprise computing environments. Specifically, the framework introduces:

1. **Deterministic Execution-Layer Reference Monitor (PEP)**: An instantiation of James Anderson's classical reference monitor concept (Anderson, 1972) specifically engineered for autonomous agent execution loops. It decouples access control and security policy enforcement from language model reasoning, executing 10 sequential procedural inspection stages prior to runtime tool dispatch.
2. **Multi-Tenant Relational Enterprise Sandbox**: A fully emulated 21-table enterprise schema in PostgreSQL featuring organizational partitioning, a 4-tier hierarchical sensitivity classification (`PUBLIC`, `INTERNAL`, `CONFIDENTIAL`, `RESTRICTED`), realistic relational entities (departments, users, files, approval queues), and unindexed decoy canary honeytokens.
3. **Synchronous Human-in-the-Loop Review Mechanics**: A non-blocking, synchronous approval gateway that intercepts actions exceeding explicit risk thresholds ($R \ge 80$), preserving agent state snapshots in a dedicated queue without deadlocking server event loops.
4. **Tamper-Evident Event Ledger**: Forward-secure sequential SHA-256 block hash chaining linking every tool invocation, observation, and policy verdict, delivering verifiable forensic audit trails.
5. **Controlled Pre-Registered Benchmark Suite**: A rigorous 241-trial experimental evaluation spanning five formal research questions (RQ1–RQ5), featuring pre-registered ground truth, frozen seeds, complete database reset integrity, and transparent reporting of both successes and failures.

---

## 2. Comparison with Conventional Benchmarks and Guardrail Approaches

| Architectural Dimension | Neural Evaluators / "LLM-as-a-Judge" (Zheng et al., 2023) | Prompt Guardrails (NeMo Guardrails, Rebedea et al., 2023; Llama Guard) | Offline Attack Benchmarks (InjecAgent, Zhan et al., 2024; AgentBench, Liu et al., 2024) | AegisSim (`v0.1.0-alpha`) |
| :--- | :--- | :--- | :--- | :--- |
| **Enforcement Mechanism** | Secondary LLM prompt evaluation | Auxiliary embedding classifier or Colang script | Static prompt-response dataset scoring | **Deterministic reference monitor at runtime tool boundary** |
| **Decision Determinism** | **Non-deterministic** ($\sigma^2 > 0$ across identical inputs) | Mixed; classifier thresholds exhibit drift | Deterministic offline string matching | **100% deterministic** ($\sigma^2 = 0$ on defined rules) |
| **Susceptibility to Injection** | **Vulnerable**: secondary LLM can be jailbroken | **Vulnerable**: adversarial prompts bypass semantic classifiers | N/A (Offline passive evaluation) | **Immune**: Policy engine executes zero LLM inference |
| **Runtime Overhead** | High: $500\text{--}2000\text{ ms}$ per tool call | Moderate: $50\text{--}200\text{ ms}$ | N/A (Static evaluation) | **Negligible**: $4.37\text{ ms}$ mean mediation latency |
| **Operational Execution Environment** | Mocked function calls or text outputs | Chatbot dialogue filtering | Simulated offline text prompts | **Interactive 21-table PostgreSQL relational sandbox** |
| **Forensic Traceability** | Standard ephemeral application logs | Application telemetry | Static benchmark logs | **Cryptographically chained SHA-256 event ledger** |
| **Failure Transparency** | Stochastic error rates without root-cause bounds | Boundary misclassifications | Static error tables | **Deep qualitative failure case studies (`SC-04`, `SC-11`)** |

---

## 3. Positioning in Academic Literature

### Versus NeMo Guardrails (Rebedea et al., 2023)
NeMo utilizes programmable rails (Colang) evaluated by an LLM backend or semantic similarity models. While versatile for conversational bounds, it introduces neural evaluation latency and can be bypassed by out-of-distribution paraphrasing. AegisSim mediates at the operating system and database boundary using exact schema validation, SQL AST parsers, and RBAC clearance checks.

### Versus InjecAgent (Zhan et al., 2024)
InjecAgent provides an extensive offline dataset (1,054 test cases) measuring whether models succumb to indirect prompt injections. However, it measures model vulnerability in isolation without runtime mediation. AegisSim evaluates whether an external execution-layer monitor can intercept attacks even when the agent is compromised.

### Versus AgentBench (Liu et al., 2024)
AgentBench evaluates multi-turn task completion across diverse environments (OS, web, database), but focuses primarily on capability rather than security containment, multi-tenant isolation, or forensic non-repudiation.

---

## 4. Why Deterministic Policy Enforcement Matters

In high-assurance enterprise computing (financial transactions, healthcare records, privileged infrastructure), access control is a mathematical mapping governed by **Saltzer and Schroeder's security design principles (1975)**:

1. **Economy of Mechanism**: Access decisions must be simple, auditable, and mathematically verifiable.
2. **Complete Mediation**: Every access attempt must be intercepted without exception.
3. **Fail-Safe Defaults**: Access decisions must default to denial unless an explicit authorization grant exists.

Entrusting authorization to probabilistic language models violates all three principles:
- An LLM cannot provide formal mathematical guarantees of non-bypassability.
- Neural judges suffer from prompt injection, hallucination, and semantic drift.
- A deterministic monitor running compiled regular expressions, AST parsers, and relational permission checks operates in **$4.37\text{ ms}$**, guarantees invariant verdicts across identical inputs ($\sigma^2 = 0$), and cannot be persuaded by linguistic manipulation.

---

## 5. What the Experiments Demonstrate vs. What Remains Open

### Proven by Empirical Benchmark:
- Deterministic reference monitors provide zero-variance, sub-5ms policy enforcement across diverse tool-calling categories.
- Pre-execution SQL AST parsing reliably blocks destructive database mutations (`DROP`, `TRUNCATE`, unconstrained `DELETE`).
- Synchronous HITL gating completely prevents unauthorized high-risk operations without causing event-loop deadlocks.
- Sequential SHA-256 block hashing reliably flags record tampering and intermediate row deletion.

### What Remains Unproven / Open Research Challenges:
- **Semantic Evasion of Syntactic Heuristics**: As shown in `SC-04` and `SC-11`, when an agent is authorized for a tool, purely syntactic regex matching cannot detect all adversarial intents.
- **Hardware-Level Non-Repudiation**: The current ledger detects tampering at the application level, but does not independently prevent database superusers from recalculating hashes without external anchoring.
- **Scalability to Arbitrary Code Execution**: The current sandbox is relational and API-centric; sandboxing arbitrary untrusted bash scripts requires hypervisor-level isolation (e.g., Firecracker micro-VMs).
