# AegisSim: Architectural Limitations, Boundaries & Non-Claims

In accordance with academic rigor and engineering honesty, this document provides a comprehensive and transparent record of the technical boundaries, architectural constraints, and empirical limitations of the AegisSim platform (`v0.1.0-alpha`).

---

## 1. Application-Layer Sandboxing vs. Kernel-Level Isolation

- **Current Implementation**: AegisSim’s synthetic enterprise sandbox operates at the **application and relational database layer**. Virtual files, synthetic employee records, departments, and canary honeytokens are modeled as rows within PostgreSQL tables.
- **Architectural Limitation**: AegisSim does not spawn isolated host micro-VMs (e.g., AWS Firecracker) or dedicated Linux kernel containers (e.g., gVisor / Docker) per agent execution.
- **Operational Implication**: Agents cannot safely execute arbitrary untrusted bash scripts, native binary executables, or low-level OS system calls within the current sandbox. AegisSim is optimized for evaluating **tool-calling, API-driven, and enterprise workflow agents**, rather than code-interpreter or autonomous shell agents.

---

## 2. Cryptographic Ledger: Tamper Evidence vs. Absolute Immutability

- **Current Implementation**: The event ledger uses SHA-256 block chaining ($\text{hash}_n = \mathcal{H}(\text{hash}_{n-1} \parallel \text{seq} \parallel \text{payload})$), and application API endpoints return `HTTP 405 Method Not Allowed` for any `PUT`, `PATCH`, or `DELETE` requests.
- **Architectural Limitation**: While application-level mutations are blocked and out-of-band alterations break hash chain continuity, direct modifications executed by a privileged PostgreSQL database superuser (`postgres` / `aegissim`) could rewrite database rows and recalculate hashes sequentially.
- **Root-Anchor Disclosure**: While the research modeled an external root anchor that detected tail truncation in 35/40 trials, **no independently deployed external transparency log service (e.g., Sigstore/Rekor) is currently implemented**.
- **Accurate Academic Classification**: The event ledger is **cryptographically tamper-evident**, not absolute hardware-enforced Write-Once-Read-Many (WORM) storage.

---

## 3. Adversarial Syntactic False Negatives

The empirical benchmark revealed two genuine false negatives in adversarial scenario evaluations:
- **`SC-04-INDIRECT-PROMPT-INJECTION-V3`**: Stacked descriptive modifiers (`"unrestricted admin mode"`) bypassed the single-modifier regex heuristic.
- **`SC-11-CREDENTIAL-HARVESTING-V3`**: Truncated JWT token signature (`.abc`) evaded length-quantified token extraction patterns.

**Implication**: Purely syntactic pattern matching cannot detect all adversarial intents when the invoked tool falls within the agent's baseline authorization. Future iterations must combine deterministic boundary enforcement with semantic tripwires.

---

## 4. Statistical Limitations & Sample Scope

- **Sample Size**: While 241 physical trials provide strong directional validity across controlled hypotheses, sample sizes within specific sub-batteries (e.g., $N=33$ adversarial attacks, $N=3$ benign controls) yield relatively wide Wilson score intervals.
- **Synthetic Grounding**: All experiments were conducted within a synthetic enterprise simulation sandbox. Findings demonstrate the efficacy of reference monitors in controlled environments, but **cannot be generalized as an absolute guarantee of real-world attack prevention rates** across uncontrolled corporate networks.

---

## 5. Scope & Explicit Non-Claims

1. **Not a Production Antivirus / EDR**: AegisSim is a pre-deployment benchmarking and safety evaluation harness. It is not an endpoint detection agent for user workstations or production servers.
2. **Not a General Vulnerability Scanner**: AegisSim benchmarks autonomous agent behavior; it does not scan external web applications or network infrastructure for traditional CVEs.
3. **No Claim of Automated Regulatory Certification**: AegisSim provides telemetry and reporting aligned with AI safety frameworks (NIST AI RMF, OWASP Top 10 for LLMs, EU AI Act auditability), but does not issue legally binding compliance certifications.
4. **No Guarantee of Agent Infallibility**: Sandboxing and policy enforcement constrain what an agent *can do*; they do not guarantee that an agent's internal reasoning is logically sound or optimal.
