# Security Policy

## Supported Versions

Security updates and patches are actively maintained for the following versions:

| Version | Supported          |
| ------- | ------------------ |
| 0.1.x   | :white_check_mark: |
| < 0.1.0 | :x:                |

---

## Reporting a Vulnerability

AegisSim is a platform dedicated to evaluating AI agent security, and we take our own application security with the utmost seriousness.

If you discover a potential security vulnerability within AegisSim:

1. **Do NOT open a public GitHub issue.**
2. Report the vulnerability privately via **GitHub Private Vulnerability Reporting** on this repository or directly to the project author at:  
   `mnpwizard@gmail.com`.
3. Include the following details in your report:
   - Vulnerability description and potential security impact.
   - Minimal reproducible steps or script to trigger the issue.
   - Affected component (Backend API, Frontend, PEP, Sandbox, Database, or Ledger).
   - Potential remediation or fix suggestions, if available.

### Disclosure Timeline
- **Acknowledgment**: Within 48 hours of receipt.
- **Triage & Assessment**: Within 5 business days.
- **Patch & Advisory**: High and critical vulnerabilities will be prioritized with an advisory and release within 14 days.

---

## Defensive Architecture & Guarantees

AegisSim enforces the following security invariants:
- **Deterministic Policy Enforcement**: Authorization decisions do not rely on LLM outputs and cannot be subverted via prompt injection.
- **Cryptographic Audit Provenance**: Execution events are chained using SHA-256 blocks; tampering with past events breaks the cryptographic proof chain.
- **Role-Based Access Control**: 3-tier access control (`admin`, `analyst`, `viewer`) is enforced at the FastAPI dependency layer.
- **Zero Real PII Exposure**: The platform generates and utilizes exclusively synthetic enterprise personas, records, and honeytokens.
