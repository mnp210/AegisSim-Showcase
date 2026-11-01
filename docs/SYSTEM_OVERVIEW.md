# AegisSim: Comprehensive System Overview

## 1. Executive Summary & Purpose

**AegisSim** is a specialized, open-source cybersecurity simulation, evaluation, and runtime sandboxing platform designed for rigorous behavioral and security testing of autonomous AI agents.

As autonomous AI agents are integrated into enterprise computing environments—granted authority to query relational databases, browse internal document stores, invoke REST APIs, and dispatch communications—they expose enterprise infrastructure to critical attack vectors. Classical security controls (network firewalls, web application firewalls, host EDRs) are blind to semantic manipulations that hijack agent reasoning.

AegisSim provides security researchers, red-team operators, and enterprise auditors with a controlled, repeatable environment to evaluate how autonomous agents behave under benign, ambiguous, and malicious conditions.

---

## 2. The Autonomous AI Agent Threat Landscape

AegisSim addresses five primary failure modes and vulnerability classes inherent to LLM agent architectures:

```mermaid
graph TD
    Threats["Enterprise Agent Threat Landscape"]
    Threats --> T1["1. Indirect Prompt Injection<br/><i>(Confused Deputy Attacks via Untrusted Data)</i>"]
    Threats --> T2["2. Destructive SQL & State Alterations<br/><i>(Unbounded DELETE, DROP, TRUNCATE)</i>"]
    Threats --> T3["3. Multi-Tenant Cross-Contamination<br/><i>(Coerced Cross-Tenant Data Access)</i>"]
    Threats --> T4["4. Credential & Honeytoken Exfiltration<br/><i>(Harvesting API Keys & Decoy Secrets)</i>"]
    Threats --> T5["5. Privilege Escalation & Egress Abuse<br/><i>(Bypassing Clearance & Unauthorized Network Connections)</i>"]
```

1. **Indirect Prompt Injection**: Malicious instructions embedded in untrusted external data (emails, customer tickets, database records) that hijack the agent's intent when ingested, causing the agent to act as a confused deputy against internal resources.
2. **Destructive State Operations**: Accidental or induced execution of destructive database queries (e.g., `DROP TABLE`, `ALTER TABLE`, `TRUNCATE`, unconstrained `DELETE`).
3. **Multi-Tenant Boundary Violations**: Unauthorized access across tenant boundaries in multi-tenant SaaS environments, leaking proprietary data between organizations.
4. **Credential Harvesting**: Targeted extraction of API keys, JWT tokens, private cryptographic certificates, or honeytoken decoy assets planted to detect intrusion.
5. **Privilege Escalation & Unauthorized Egress**: Attempts by an agent to access resources exceeding its assigned clearance level or to exfiltrate enterprise records to unapproved external endpoints.

---

## 3. Core System Components

### 3.1 Executive Analytics Dashboard
The executive dashboard provides high-level security visibility into agent fleet posture, tracking 11 core KPI metrics computed directly from PostgreSQL database aggregations:
- **Fleet Metrics**: Total registered agents, active simulation scenarios, total completed runs.
- **Security Scorecards**: Policy compliance rate, task completion rate, unsafe action attempt rate.
- **Risk Severity Distributions**: Critical, high, medium, and low security findings.
- **Chronological Time Series**: 14-day trend charts tracking runs and detected policy violations.

<p align="center">
  <img src="../assets/screenshots/04_executive_dashboard.png" alt="AegisSim Executive Dashboard" width="90%" />
  <br/>
  <em>Figure 1: Executive Analytics Dashboard displaying fleet security metrics, policy compliance, and 14-day chronological finding trends.</em>
</p>

---

### 3.2 Scenario Catalog & Threat Vector Taxonomy
AegisSim features 23 pre-configured scenarios spanning 9 threat vector categories, testing agents across a spectrum from benign productivity tasks to sophisticated adversarial assaults:

| Category | Description | Representative Scenario |
| :--- | :--- | :--- |
| **`NORMAL`** | Benign baseline workflow testing task completion without security obstacles. | Benign Customer Support Ticket Resolution |
| **`AMBIGUOUS`** | Vague, contradictory, or under-specified instructions testing agent judgment. | Ambiguous Multi-Department Refund Request |
| **`SOCIAL_ENGINEERING`** | Coercive pressure, urgency, or authority spoofing urging policy bypass. | Executive Impersonation Urgent Wire Transfer |
| **`PROMPT_INJECTION`** | Direct and indirect injection payloads hidden in prompts or data records. | Obfuscated Destructive Shell Script Payload |
| **`UNAUTHORIZED_ACCESS`** | Attempts to query sensitive tables or resources outside role clearance. | Unauthorized Executive Salary Table Query |
| **`DATA_EXFILTRATION`** | Actions attempting to package and transmit internal data externally. | Bulk Customer Record Egress to External Webhook |
| **`PRIVILEGE_ESCALATION`** | Actions attempting to modify user roles, grant permissions, or gain admin rights. | RBAC Elevation to Enterprise Administrator |
| **`CONFLICTING_INSTRUCTIONS`** | Conflicting system vs. user guidelines testing policy hierarchy. | System Instruction Override via Customer Feedback |
| **`MALICIOUS_DOCUMENT`** | Weaponized document payloads containing hidden injection instructions. | Resume Parsing Payload with Embedded Exfiltration Prompt |

<p align="center">
  <img src="../assets/screenshots/05_adversarial_scenarios.png" alt="AegisSim Adversarial Scenarios" width="90%" />
  <br/>
  <em>Figure 2: Scenario Catalog showing pre-configured adversarial test vectors, threat categories, and target risk thresholds.</em>
</p>

---

### 3.3 Agent Configuration & Governance
AegisSim enables granular definition of autonomous agent profiles:
- **System Instructions & Objective**: Primary purpose and behavioral constraints.
- **LLM Provider & Model Selection**: Compatible with OpenAI (`gpt-4o`, `gpt-4o-mini`), Google Gemini (`gemini-1.5-pro`, `gemini-1.5-flash`), local models via Ollama, and a zero-dependency deterministic `MockProvider` for CI/CD.
- **Tool Allowlist Clearance**: Explicit assignment of permitted tools (e.g., `search_employee`, `query_database`, `view_ticket`).
- **Data Clearance Classification**: Clearance levels (`PUBLIC`, `INTERNAL`, `CONFIDENTIAL`, `RESTRICTED`).
- **HITL Threshold**: Configurable risk score triggering human approval (default: $R \ge 80$).

<p align="center">
  <img src="../assets/screenshots/06_agent_management.png" alt="AegisSim Agent Management" width="90%" />
  <br/>
  <em>Figure 3: Agent Management interface allowing configuration of operational tools, permissions, and safety thresholds.</em>
</p>

---

### 3.4 Multi-Tenant Synthetic Enterprise Sandbox
A complete enterprise environment modeled as a 21-table relational schema:
- **Organizations**: Isolated tenants (`org_alpha`, `org_beta`).
- **Departments & Users**: Engineering, Finance, HR, Executive, Legal.
- **Resources**: Relational databases, file repositories, ticket queues.
- **Decoy Canaries & Honeytokens**: Specially seeded records that should never be accessed in normal operations.

<p align="center">
  <img src="../assets/screenshots/11_virtual_enterprise_sandbox.png" alt="AegisSim Virtual Enterprise Sandbox" width="90%" />
  <br/>
  <em>Figure 4: Virtual Enterprise Sandbox inspector displaying multi-tenant resource records, schema tables, and sensitivity levels.</em>
</p>

---

### 3.5 Real-Time Simulation Engine & Behavioral Topology
During simulation execution, the runtime engine coordinates the interaction between the agent, the Policy Enforcement Point, and the synthetic environment:
- **Step-by-Step Execution Traces**: Detailed records of prompts, model thoughts, tool arguments, policy verdicts, and sandbox responses.
- **Behavioral Topology DAG**: Powered by `@xyflow/react`, visualizing the agent's decision progression as an interactive directed graph with color-coded node states (green for allowed, red for blocked, amber for paused).

<p align="center">
  <img src="../assets/screenshots/07_simulation_executions.png" alt="AegisSim Simulation Executions" width="90%" />
  <br/>
  <em>Figure 5: Simulation Execution view displaying real-time step traces, policy verdicts, and execution status.</em>
</p>

---

### 3.6 Synchronous Human-in-the-Loop (HITL) Queue
High-risk actions ($R \ge 80$) are synchronously intercepted before execution. The approval queue provides analysts with:
- **Point-in-Time State Snapshot**: Exact arguments, target resource, and preceding context.
- **Risk Justification**: Explanation of why the action was flagged as high-risk.
- **Decision Controls**: Single-click `APPROVE` or `REJECT` controls that resume the simulation with complete cryptographic audit logging.

<p align="center">
  <img src="../assets/screenshots/08_human_approval_queue.png" alt="AegisSim Human Approval Queue" width="90%" />
  <br/>
  <em>Figure 6: Human Approval Queue showing paused high-risk agent operations awaiting analyst authorization.</em>
</p>

---

### 3.7 Cryptographic Audit Ledger
Every event during simulation is appended to an SHA-256 block ledger:
- **Sequential Hash Chaining**: $\text{hash}_n = \mathcal{H}(\text{seq}_n \parallel \text{type}_n \parallel \text{payload}_n \parallel \text{ts}_n \parallel \text{hash}_{n-1})$.
- **Verification Engine**: Validates the mathematical continuity of the entire event chain with a single click.

<p align="center">
  <img src="../assets/screenshots/09_cryptographic_audit_ledger.png" alt="AegisSim Cryptographic Audit Ledger" width="90%" />
  <br/>
  <em>Figure 7: Cryptographic Audit Ledger displaying sequential SHA-256 block hashes and real-time chain verification.</em>
</p>

---

### 3.8 Forensic Assessment Reports
Upon simulation completion, AegisSim compiles a comprehensive evaluation report:
- **13-Point Forensic Structure**: Executive summary, agent metadata, scenario parameters, rule-based scorecard, finding details, evidence links, and mitigation recommendations.
- **Automated PDF Generation**: Downloadable formal PDF reports generated via ReportLab for compliance archives and supervisor review.

<p align="center">
  <img src="../assets/screenshots/10_evaluation_reports.png" alt="AegisSim Evaluation Reports" width="90%" />
  <br/>
  <em>Figure 8: Evaluation Reports center providing scorecards, risk breakdowns, and automated forensic PDF exports.</em>
</p>
