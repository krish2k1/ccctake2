# Requirements Register — Session II (Triage & Vendor Match Layer)

Numbering convention: **R-{theme}-{n}**. Themes: CAT (catalog), TRIG (trigger / service identification), POOL (vendor pool / eligibility), SCORE (scoring), MATCH (match algorithm), UX (adjuster experience), LOOP (learning loop), INTG (integration), COMP (compliance), NFR (non-functional), RISK (risk mitigations).

---

## Functional — Service Catalog

| ID | Requirement |
|---|---|
| R-CAT-01 | Maintain a canonical Service Catalog of 36 services across 6 families (Medical Direction, Medical Spend, RTW & Indemnity, Investigation & Defense, Resolution & Settlement, Records & Admin). |
| R-CAT-02 | Each service record contains: id, name, family, typical trigger conditions, typical timing window, typical unit cost band, dependent / prerequisite services, regulatory notes per state. |
| R-CAT-03 | Catalog changes (add / retire a service) require operations + compliance approval and are versioned. |
| R-CAT-04 | Catalog is queryable by trigger, family, jurisdiction, and dependency. |

## Functional — Service Identification (Triggers)

| ID | Requirement |
|---|---|
| R-TRIG-01 | Service identification is hybrid: deterministic rules for unambiguous services, multi-label classifier for discretionary services, timing model for time-sensitive services. |
| R-TRIG-02 | Surveillance (D2) recommendation requires explicit human approval before vendor engagement, regardless of model confidence. |
| R-TRIG-03 | Deterministic vs probabilistic classification is declared per service in the catalog. |
| R-TRIG-04 | Triggers consume the canonical claim snapshot: FNOL data, severity score, litigation propensity score, NCCI body part + nature, jurisdiction, language indicator, geography, current reserve, engaged services so far, treatment plan, opioid duration, lag time, treating provider score, catastrophic flags. |
| R-TRIG-05 | Trigger thresholds are configurable per service and revisited monthly with ROI evaluation. |
| R-TRIG-06 | Service identification re-runs whenever a claim event materially changes the snapshot (event-driven, not polled). |
| R-TRIG-07 | The classifier provides per-recommendation feature attribution (SHAP-style) suitable for adjuster display. |

## Functional — Vendor Pool & Eligibility

| ID | Requirement |
|---|---|
| R-POOL-01 | Vendor Master is the canonical source of vendor identity, services offered, jurisdictions, credentials, capacity, rate card, contacts. |
| R-POOL-02 | Eligibility is gated by 8 hard criteria: (1) state licensure/credentials current, (2) E&O insurance at carrier-required limits, (3) BAA executed, (4) compliance attestations on file (AI, surveillance laws, MSA registration as applicable), (5) no active regulatory action, (6) SOC 2 Type II or equivalent, (7) machine-readable rate card on file, (8) capacity disclosed and below carrier ceiling. |
| R-POOL-03 | Eligibility is checked at runtime; a vendor that lapses any gate is removed from the candidate pool until reinstated. |
| R-POOL-04 | Per-claim filter applies: jurisdiction of claim, conflict-of-interest exclusions (claim party, opposing counsel relationship), explicit claim-level exclusion list. |
| R-POOL-05 | Vendor capacity utilization is updated at least daily; no vendor is routed to above 85% of disclosed weekly capacity. |
| R-POOL-06 | Vendor onboarding workflow enforces the 8 gates with a documented evidence artifact for each. |

## Functional — Vendor Scoring

| ID | Requirement |
|---|---|
| R-SCORE-01 | Composite score = 0.40·Outcome + 0.25·Cost + 0.15·Speed + 0.10·SpecialtyFit + 0.05·AdjusterSat + 0.05·Compliance. Weights configurable per service family within governance. |
| R-SCORE-02 | Outcome scoring uses propensity-weighted estimation to correct for assignment bias (top vendors get easier claims). |
| R-SCORE-03 | Time decay: 6 months at 60% weight, 7–18 months at 40%, beyond 18 months excluded. |
| R-SCORE-04 | Minimum sample of 30 closed engagements per service line per vendor before a published rank; below threshold, vendor is flagged (provisional). |
| R-SCORE-05 | Specialty-fit weighting is computed per-claim (body part, language, geography, venue) — not a static vendor attribute. |
| R-SCORE-06 | Adjuster satisfaction collected via short post-engagement survey; aggregate score with minimum 10 responses before published. |
| R-SCORE-07 | Compliance score includes audit findings, DOI complaints, EDI failure rate, late filings. |
| R-SCORE-08 | Scoring methodology published to vendors annually with sufficient detail to self-improve, insufficient to game; published to regulators on request. |

## Functional — Match Algorithm

| ID | Requirement |
|---|---|
| R-MATCH-01 | Match flow: (1) snapshot claim, (2) identify services, (3) retrieve eligible pool per service, (4) score and rank, (5) compose adjuster recommendation. |
| R-MATCH-02 | Output presents top-3 vendors per recommended service with composite score, factor-level breakdown, estimated cost, estimated outcome impact, and rationale. |
| R-MATCH-03 | Anti-concentration cap: no single vendor receives more than 50% of routed volume per service over a rolling 90-day window. |
| R-MATCH-04 | Exploration quota: 5% of routing volume goes to rank-4 and rank-5 eligible vendors to refresh cold-vendor data. |
| R-MATCH-05 | Every recommendation persisted with: claim snapshot inputs, model versions, factor weights, score outputs, top-3 candidates, adjuster action, downstream outcome (when known). |
| R-MATCH-06 | Recommendations are immutable once issued; successor recommendations on new events link to predecessors by lineage. |
| R-MATCH-07 | The Recommendation Composer publishes an event on issuance and on adjuster action (accept / swap / reject). |

## Functional — Adjuster Experience

| ID | Requirement |
|---|---|
| R-UX-01 | Recommendations are embedded in the adjuster workflow at the relevant decision moment (e.g., NCM-referral screen shows NCM recommendation), not delivered to a separate inbox. |
| R-UX-02 | Display includes factor-level breakdown, comparable peer claims, and an estimated cost / outcome impact band for each candidate. |
| R-UX-03 | Override path is 2 clicks maximum; override reason is selected from a controlled vocabulary plus optional free text. |
| R-UX-04 | Override patterns are trended by service, by adjuster team, by jurisdiction; surfaced to the model team as features-to-investigate. |
| R-UX-05 | For the first 12 months of rollout, individual adjuster override rate is not used in performance reviews. |
| R-UX-06 | Catastrophic-indicator services (A3, A5 escalations) are auto-presented at FNOL+24h, not waiting for adjuster review. |

## Functional — Learning Loop & Attribution

| ID | Requirement |
|---|---|
| R-LOOP-01 | Vendor score refresh runs weekly. |
| R-LOOP-02 | Service-identifier classifier retrains quarterly with holdout validation and documented bias testing per the model risk policy. |
| R-LOOP-03 | Trigger threshold calibration reviewed monthly with ROI-curve analysis. |
| R-LOOP-04 | Override learning reviewed monthly; clustered override patterns trigger feature investigation. |
| R-LOOP-05 | 5% randomized holdout cohort during first 6 months of production rollout to enable counterfactual savings attribution. |
| R-LOOP-06 | Quarterly savings attribution published to finance using propensity-score-matched estimator. |
| R-LOOP-07 | Drift monitoring on input feature distributions; retraining triggered automatically on threshold breach. |

## Integration

| ID | Requirement |
|---|---|
| R-INTG-01 | Inbound events consumed from claims core (FNOL filed, severity scored, treatment plan submitted, reserve changed, status changed, NCM referral requested, defense counsel requested). |
| R-INTG-02 | Outbound vendor engagement via EDI / Vendor API; intake packet generated per service; capacity decremented; SLA timer started. |
| R-INTG-03 | Vendor API specification published and versioned; degraded path (PDF intake, manual outcome entry) supported for vendors not on the API. |
| R-INTG-04 | Outcome events ingested from vendor (cycle time, services rendered, invoice line items) and from claim outcome (RTW date, closure, total cost). |
| R-INTG-05 | Policy / premium-audit system receives writebacks where vendor work affects exposure (e.g., payroll reconciliation, misclassification finding). |
| R-INTG-06 | Existing bill-review engine, PBM platform, PPO/MPN network data, and ISO ClaimSearch are integrated read paths, not replaced. |

## Compliance (non-negotiable)

| ID | Requirement |
|---|---|
| R-COMP-T01 | No automated compensability denial; layer recommends services, never denies. |
| R-COMP-T02 | Bias testing performed on recommendation outputs (vendor steering by claimant demographic, jurisdiction, language) pre-deployment and at least annually. |
| R-COMP-T03 | Full audit trail per recommendation: snapshot, model versions, factor weights, score outputs, adjuster action, final outcome. |
| R-COMP-T04 | Vendor surveillance (D2) requires named-human approval prior to engagement; surveillance approval and rationale logged. |
| R-COMP-T05 | Jurisdiction-aware: every recommendation respects state-specific vendor eligibility (MPN compliance, certified-network rules, attorney-admission rules, surveillance laws). |
| R-COMP-T06 | Consumer notice mechanism honored where state law requires AI-use disclosure in claim handling (CO, NY currently; configurable). |
| R-COMP-T07 | Vendor due-diligence file (the 8 gates + ongoing attestations) is the regulator-ready vendor compliance artifact. |

## Non-functional

| ID | Requirement |
|---|---|
| R-NFR-01 | Recommendation latency: P50 < 2 seconds, P95 < 5 seconds from event arrival to recommendation persisted. |
| R-NFR-02 | All claim data in flight and at rest is encrypted; PHI access controlled per HIPAA / state law. |
| R-NFR-03 | Idempotent event handling: replay of an event produces the same recommendation given the same snapshot and model versions. |
| R-NFR-04 | Recommendation immutability is enforced at the persistence layer; corrections create successor records, never overwrite. |
| R-NFR-05 | Multi-jurisdiction configuration is data-driven; no jurisdiction-specific logic hard-coded outside the rules engine. |
| R-NFR-06 | Adoption metric: by end of Phase 2, ≥70% of recommendations are accepted unchanged; override-with-reason is captured on the remainder. |

## Risk mitigations carried forward

| ID | Risk | Mitigation requirement |
|---|---|---|
| RM-01 | Disparate-impact in vendor steering | R-COMP-T02 bias testing on outputs |
| RM-02 | Surveillance privacy exposure | R-COMP-T04 named-human approval |
| RM-03 | Anti-competitive steering claim | R-SCORE-08 published methodology + R-MATCH-04 exploration quota |
| RM-04 | Adjuster rejection / cultural fail | R-UX-05 no individual performance use Y1 + R-UX-01 embedded UX |
| RM-05 | Vendors gaming the score | R-SCORE-02 outcomes the vendor can't fabricate + R-LOOP-04 override learning |
| RM-06 | Top vendor capacity collapse | R-MATCH-03 anti-concentration + R-POOL-05 capacity ceiling |
| RM-07 | Causal bias in outcome scoring | R-LOOP-05 5% holdout |
| RM-08 | Concept drift on injury patterns | R-LOOP-07 drift monitoring |
| RM-09 | Cold-start on new vendors/services | R-MATCH-04 exploration quota + R-SCORE-04 flag-not-rank below threshold |
| RM-10 | Cost optimization at outcome expense | R-SCORE-01 outcome weighted highest + DOI-complaint monitoring |
