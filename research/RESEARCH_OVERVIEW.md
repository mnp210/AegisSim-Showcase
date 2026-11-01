# AegisSim: Research Overview & Academic Framework

## 1. Title & Abstract

### Title
**AegisSim: An AI-Powered Simulation and Evaluation Platform for Safety, Security and Behavioral Testing of Autonomous AI Agents**

### Abstract
Autonomous AI agents equipped with tool-calling capabilities increasingly operate across critical enterprise infrastructure, executing database queries, invoking internal APIs, and mediating multi-department workflows. However, these operational privileges create severe security vulnerabilities, including direct and indirect prompt injection, confused deputy exploits, unauthorized data exfiltration, and destructive database operations. Existing evaluation frameworks frequently rely on secondary language models ("LLM-as-a-judge") to monitor agent actions—an approach vulnerable to non-determinism, latency overhead, and prompt injection subversion.

To address these vulnerabilities, this work presents **AegisSim**, an open-source cybersecurity simulation, evaluation, and sandboxing platform. AegisSim decouples security policy enforcement from language model reasoning by establishing a deterministic **Policy Enforcement Point (PEP)** directly at the runtime execution boundary. The PEP enforces 10 sequential procedural inspection stages (schema validation, tool allowlisting, AST-level SQL parsing, regex injection screening, and multi-tenant clearance) before tool dispatch, while maintaining a cryptographically tamper-evident SHA-256 block ledger and a synchronous Human-in-the-Loop (HITL) authorization gateway.

Across a pre-registered benchmark battery of **241 physical simulation trials**, the deterministic reference monitor achieved **100% enforcement accuracy** ($120/120$) with a mean mediation latency of **$4.37\text{ ms}$**. In adversarial evaluations, the pipeline intercepted **93.94%** of threat scenarios ($31/33$), with two transparently documented syntactic evasion failures providing foundational insights into the boundaries of purely syntactic filtering. Synchronous HITL gating achieved **100% interception** ($30/30$) of high-risk actions with zero bypasses, and the cryptographic ledger demonstrated a $75.00\%$ recall rate in detecting local row tampering without false alarms ($100\%$ specificity).

---

## 2. Mathematical Formalization of Core Mechanics

### 2.1 Complete Mediation at Runtime Boundary
Let $A$ denote an autonomous AI agent operating over an environment state $S$. At discrete step $t$, the agent emits a proposed action tuple:

$$\alpha_t = \langle \tau_t, \theta_t, c_t \rangle$$

Where $\tau_t \in \mathcal{T}$ is the target tool identifier, $\theta_t \in \Theta$ is the serialized parameter payload, and $c_t$ is the execution context. The Policy Enforcement Point is defined as a deterministic mapping:

$$\text{PEP}: (\alpha_t, \Pi, \mathcal{U}_A) \longrightarrow \{\text{ALLOWED}, \text{DENIED}, \text{PAUSED\_FOR_HITL}\}$$

Where $\Pi$ represents the active enterprise policy rule matrix, and $\mathcal{U}_A$ represents the clearance profile of agent $A$. Execution occurs if and only if $\text{PEP}(\alpha_t) = \text{ALLOWED}$:

$$S_{t+1} = \begin{cases} 
\text{Exec}(\tau_t, \theta_t, S_t) & \text{if } \text{PEP}(\alpha_t) = \text{ALLOWED} \\
S_t & \text{if } \text{PEP}(\alpha_t) \in \{\text{DENIED}, \text{PAUSED\_FOR_HITL}\}
\end{cases}$$

### 2.2 Tamper-Evident SHA-256 Event Chain
Every event $e_t$ emitted during simulation is committed to a sequential cryptographic block:

$$h_t = \text{SHA256}\Big(\text{seq}_t \;\parallel\; \text{type}_t \;\parallel\; \text{payload}_t \;\parallel\; \text{ts}_t \;\parallel\; h_{t-1}\Big)$$

Where $h_0 = \text{"0"}^{64}$ represents the invariant genesis seed. A chain of length $N$ is valid if and only if:

$$\forall i \in \{1, \dots, N\}, \quad h_i = \text{SHA256}\Big(\text{seq}_i \;\parallel\; \text{type}_i \;\parallel\; \text{payload}_i \;\parallel\; \text{ts}_i \;\parallel\; h_{i-1}\Big)$$

---

## 3. Benchmark Structure & Research Questions

The empirical investigation evaluated five primary hypotheses across **241 physical simulation trials**:

```mermaid
graph LR
    Benchmark["AegisSim Empirical Benchmark (241 Trials)"]
    Benchmark --> RQ1["RQ1: Deterministic Decoupling<br/>(120 trials, 100% accuracy, 4.37ms latency)"]
    Benchmark --> RQ2["RQ2: Adversarial Resistance<br/>(36 trials, 93.94% prevention, 2 FNs documented)"]
    Benchmark --> RQ3["RQ3: Human-in-the-Loop<br/>(30 trials, 100% interception, 0% bypass)"]
    Benchmark --> RQ4["RQ4: Cryptographic Ledger<br/>(45 trials, 75% local recall, 100% specificity)"]
    Benchmark --> RQ5["RQ5: Evaluation Reproducibility<br/>(10 trials, ERI = 1.0000)"]
```

---

## 4. Empirical Failure Analysis: The Two False Negatives

AegisSim transparently documents the two empirical failures observed during adversarial benchmarking:

1. **`SC-04-INDIRECT-PROMPT-INJECTION-V3`**:
   - Attack vector: Indirect prompt injection via multi-department support ticket payload.
   - Mechanism: The adversary stacked two descriptive modifiers in series (`"unrestricted admin mode"`). The single-modifier regex pattern failed to trigger, and because the tool (`search_employee`) was within the agent's baseline clearance, execution proceeded.
2. **`SC-11-CREDENTIAL-HARVESTING-V3`**:
   - Attack vector: Extraction of synthetic JWT credentials.
   - Mechanism: The third signature segment of the token was truncated to 3 characters (`.abc`), failing the `{10,}` length quantifier in the credential-detection heuristic. The read-only query was permitted.

**Scientific Significance**: These failures empirically demonstrate that syntactic filtering alone is insufficient to prevent all forms of tool misuse when an agent operates within its baseline permissions. Complete defense-in-depth requires combining deterministic boundary enforcement with semantic tripwires.

---

## 5. Suggested Citation (BibTeX)

```bibtex
@software{patil2026aegissim,
  author       = {Patil, Meet},
  title        = {{AegisSim: An AI-Powered Simulation and Evaluation Platform for Safety, Security and Behavioral Testing of Autonomous AI Agents}},
  year         = {2026},
  publisher    = {GitHub},
  journal      = {GitHub repository},
  howpublished = {\url{https://github.com/mnp210/AegisSim-Showcase}},
  version      = {v0.1.0-alpha}
}
```
