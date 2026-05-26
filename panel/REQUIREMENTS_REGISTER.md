# Requirements Register
## Platform for LAE Optimization — Mid-Size Monoline WC Carrier

---

## Business Objectives

| ID | Objective | Measure | Target |
|---|---|---|---|
| BO-1 | Reduce blended LAE ratio | LAE / Earned Premium | -2.0 to -2.5 pts over 36 months |
| BO-2 | Reduce ULAE specifically | ULAE / Earned Premium | From ~7% to ~5.5% |
| BO-3 | Reduce ALAE specifically | ALAE / Earned Premium | From ~4% to ~3% |
| BO-4 | Improve adjuster effective capacity | Open claims per adjuster (quality-weighted) | +25% capacity without quality loss |
| BO-5 | Improve vendor cost discipline | Vendor spend per closed claim | -15% on managed vendor categories |

---

## Functional Requirements

### F1 — Intake & Intelligence
- **F1.1** Ingest FNOL from all channels (phone, web, email, EDI from employers, MGA portals).
- **F1.2** Apply document AI to extract structured fields from medical records, wage statements, and FNOL artifacts.
- **F1.3** Produce an early severity score within 24 hours of FNOL using available features.
- **F1.4** Route claims to a recommended desk/queue based on severity, jurisdiction, complexity.

### F2 — Triage & Assignment
- **F2.1** Maintain dynamic caseload balance across adjuster desks.
- **F2.2** Trigger nurse case management referral when duration or severity indicators cross threshold (target: by day 14 for qualifying claims).
- **F2.3** Escalate claims showing drift from expected trajectory.

### F3 — Medical Management
- **F3.1** Run real-time anomaly detection on medical bills (upcoding, unbundling, out-of-network).
- **F3.2** Track provider outcomes (duration, cost per claim, RTW success).
- **F3.3** Steer claimants toward higher-performing providers within network rules.
- **F3.4** Support nurse case management workflow with task assignment, documentation, and outcome tracking.

### F4 — Litigation Management
- **F4.1** Score litigation propensity within 30 days of FNOL.
- **F4.2** Recommend early settlement strategy for high-propensity claims meeting criteria.
- **F4.3** Track defense counsel performance and apply rate cards.
- **F4.4** Provide a settlement reserve range based on historical comparables.

### F5 — Vendor Orchestration
- **F5.1** Maintain a canonical vendor master with rate cards by service category.
- **F5.2** Validate invoices against contracted rates; flag exceptions.
- **F5.3** Produce a vendor performance scorecard with cost, quality, and SLA metrics.
- **F5.4** Detect billing anomalies across DME, transportation, translation, surveillance.

### F6 — Reserve & Reporting
- **F6.1** Provide actuarial reserving support: data extracts, severity-adjusted IBNR inputs, scenario analytics.
- **F6.2** Produce statutory expense allocation reports (IEE Part III consistent).
- **F6.3** Operational dashboards: LAE by category, by state, by claim cohort, by adjuster, by vendor.
- **F6.4** Executive scorecard tied to BO-1 through BO-5.

### F7 — Data & ML Platform
- **F7.1** Ingest events from claims core, policy core, billing, document repositories, vendors.
- **F7.2** Maintain a claims-centric canonical data model (policy → claim → medical bill → vendor invoice linked).
- **F7.3** Provide a feature store for production ML.
- **F7.4** Maintain a model registry with version, training data lineage, performance metrics, owner.
- **F7.5** Monitor model performance and data drift in production.

---

## Non-Functional Requirements

### Compliance & Governance
- **NF-COMP-01** Full audit trail of every AI-influenced decision (model version, inputs, outputs, human overrides, final action). _(Owner: Linda)_
- **NF-COMP-02** Pre-deployment + at-least-annual bias testing for every production model influencing claim outcomes.
- **NF-COMP-03** Human review required for compensability decisions, payment denials, and settlement authorizations above defined threshold.
- **NF-COMP-04** Jurisdiction-aware rules engine — no single-state-assumption logic.
- **NF-COMP-05** Consumer-facing AI disclosure mechanism for states that require it.
- **NF-COMP-06** Prompt pay statutes encoded and enforced in the rules engine; violations alarmed.

### Security & Privacy
- **NF-SEC-01** HIPAA/HITECH-aligned controls for medical data (encryption at rest + in transit, access logging, role-based access).
- **NF-SEC-02** BAAs with all third-party vendors handling PHI.
- **NF-SEC-03** PII minimization in non-production environments; synthetic or masked data only.

### Architecture
- **NF-ARCH-01** Event-driven integration with claims core, not a replacement of it.
- **NF-ARCH-02** Idempotent event handlers — replay-safe.
- **NF-ARCH-03** Capability-domain separation; bounded contexts.
- **NF-ARCH-04** Tech-agnostic capabilities; specific stack choices deferred to detailed design.

### Operations
- **NF-OPS-01** RPO ≤ 1 hour, RTO ≤ 4 hours for claims processing capability.
- **NF-OPS-02** 99.5% uptime for adjuster-facing surfaces (business hours).
- **NF-OPS-03** Observability: business KPIs, technical health, model performance — single pane.

### Explainability
- **NF-EXP-01** Any model influencing a money decision must be explainable per-prediction (e.g., feature contributions).
- **NF-EXP-02** Reserve estimates must remain explainable; no black-box reserve models.

---

## Out of Scope (this phase)

- Replacement of the claims core system.
- Replacement of the policy or billing systems.
- Direct claimant-facing portal (separate program).
- Underwriting expense (acquisition, G&A) optimization — addressed by a separate roadmap; this program is LAE-focused.

---

## Cross-References

- Source: [PANEL_TRANSCRIPT.md](PANEL_TRANSCRIPT.md)
- Decisions: [DECISIONS_LOG.md](DECISIONS_LOG.md)
- Open questions: [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md)
- Glossary: [GLOSSARY.md](GLOSSARY.md)
