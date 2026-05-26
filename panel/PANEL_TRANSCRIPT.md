# Expert Panel Transcript
## Topic: Building a Platform to Optimize the Expense Ratio (LAE focus) for a Mid-Size Monoline WC Carrier

**Date:** 2026-05-26
**Carrier profile anchor:** Monoline workers' comp, ~$300M–$1B DWP, multi-state (10–25 states)
**Primary lever:** Loss Adjustment Expense (LAE) — ALAE + ULAE
**Stack:** Tech-agnostic (capabilities focus)

**Moderator:** Dr. Ava Chen
**Scribe:** Marcus Reyes
**Panelists:** Rob Hutchinson (WC Domain), Dr. Priya Sharma (Actuarial), Sandy Wexler (Claims Ops), Daniel O'Brien (Underwriting), Aisha Patel (Tech Architect), Dr. James Kim (Data Science), Linda Volkov (Regulatory)

---

## §1 — Frame the problem (Moderator)

**Ava:** We have one hour. The brief is to optimize expense ratio for a mid-size monoline WC carrier. The stated lever is LAE. Before we propose capabilities, I want a number on the table. Priya — what does the LAE breakdown look like for a carrier in this band?

**Priya:** For monoline WC in the $300M–$1B DWP range, blended LAE typically runs 11–14% of premium. Top-quartile carriers are at 8–9%. Decompose that: ULAE — the salaried claims staff and overhead — is roughly 6–8 points. ALAE — defense, IME, surveillance, investigation — is 3–6 points. The gap to top quartile is 3–5 points. On a $500M book, that's $15–25M annually.

**Ava:** So the prize is $15–25M run-rate savings. Rob — does that match your operating experience?

**Rob:** It does. And I'd add: a carrier won't capture the full gap. Realistically, a well-executed three-year program can take a mid-pack carrier from, say, 12.5% to 10%. That's 2.5 points, ~$12M on a $500M book. The last point gets very hard.

**Ava:** Logged. **Target:** 2.0–2.5 point LAE reduction over 36 months. Sandy — break down the waste.

---

## §2 — Where the LAE waste sits (Sandy)

**Sandy:** From running an LAE reduction program at scale, the buckets are roughly:
- **25–30% of waste:** over-staffing on routine medical-only claims that should be straight-through or near-touchless.
- **15–20%:** vendor leakage — overlapping vendors, no rate cards, no performance accountability.
- **10–15%:** duplicate document intake — FNOL data, medical records, wage statements re-keyed multiple times.
- **10–15%:** late nurse case management — nurse engaged at day 60 when day 14 was the right call.
- **5–10%:** litigation that should have been settled earlier.
- The remainder: scattered process inefficiency.

**Ava:** Rob, you nodding?

**Rob:** That's right. I'd flip one: in some carriers vendor leakage is the bigger bucket. Depends on whether they ever cleaned up their vendor ecosystem.

**Priya:** Sandy — when you say "over-staffing on med-only," you mean a carrier paying an adjuster $85K to spend 4 hours total on a $2,500 claim. That's a ULAE-to-loss ratio that doesn't make sense.

**Sandy:** Exactly.

**Marcus (Scribe):** Capturing this as **opportunity buckets** in the register: O-1 routine claim automation, O-2 vendor leakage, O-3 document intake duplication, O-4 nurse triage timing, O-5 litigation avoidance.

---

## §3 — Required platform capabilities (open round)

**Ava:** Aisha, organize the capabilities. Everyone push back on scope.

**Aisha:** Seven capability domains, sitting *above* the claims core system — not replacing it:
1. **Intake & Intelligence** — FNOL ingestion, document AI, early severity scoring
2. **Triage & Assignment** — routing, caseload balancing, escalation
3. **Medical Management** — bill review AI, network steering, nurse case management workflow
4. **Litigation Management** — litigation prediction, defense counsel selection, settlement optimization
5. **Vendor Orchestration** — unified vendor invoicing, performance scoring, rate enforcement
6. **Reserve & Reporting** — IBNR support, expense allocation, executive dashboards
7. **Data & ML Platform** — feature store, model registry, monitoring, explainability

**James:** Endorsed. I'd add that the ML models live inside the capabilities, not as a separate "AI module." Severity score lives in Intake. Litigation propensity lives in Litigation Management. Bill anomaly detection lives in Medical Management. Otherwise you build a fancy lab that nobody uses.

**Linda:** Before we go further — I need three things baked into every capability, not bolted on:
1. **Full audit trail** of every AI-influenced decision (model version, inputs, outputs, overrides, final action).
2. **Explainability** — for any model that influences money.
3. **Human review** for any decision affecting compensability, payment denial, or settlement above a threshold.

**Ava:** Logged as cross-cutting non-functional requirements. Daniel?

**Daniel:** One more cross-cutting concern: the platform must **share data with policy and billing**, not be claims-only. If we segregate, finance can't reconcile and underwriting can't learn from claim outcomes.

**Ava:** Logged.

---

## §4 — Ranked priorities (Tier 1 / 2 / 3)

**Ava:** Sandy, force-rank by ROI.

**Sandy:** **Tier 1 (12-month payback):**
- T1.1 — Intelligent FNOL intake with severity scoring
- T1.2 — Automated medical-only claim triage (target: 30–40% of med-only at near-touchless)
- T1.3 — Document AI for medical records and FNOL artifacts
- T1.4 — Vendor performance scorecard + rate-card enforcement

**Tier 2 (18–24 months):**
- T2.1 — Litigation propensity model + early settlement workflow
- T2.2 — RTW orchestration platform
- T2.3 — Real-time medical bill review with AI anomaly detection
- T2.4 — Dynamic caseload balancing across adjuster desks

**Tier 3 (24–36 months):**
- T3.1 — Closed-loop provider scoring → network steering
- T3.2 — Vendor billing fraud detection
- T3.3 — Adjuster knowledge assistant (GenAI) for claim file Q&A

**Rob:** Push back on T1.2. "Near-touchless" is an honest goal for clean med-only claims with a single provider visit. Not for anything ambiguous. The platform must be conservative on what it routes to STP, or we will create complaints.

**Linda:** Endorsed. And the platform must never auto-deny. It can recommend; a licensed adjuster decides.

**Sandy:** Agreed. Touchless ≠ decisionless. The adjuster reviews; the platform pre-populates the file so review is a 90-second affirmation, not 90 minutes.

**Marcus (Scribe):** Capturing as **Decision D-7: No auto-denial; AI recommends, licensed adjuster decides.**

---

## §5 — Data needs (James)

**James:** To do Tier 1 well, we need:
- Linked **policy → claim → medical bill → vendor invoice** records, joined across IDs.
- Adjuster notes (free text, sometimes voice).
- Medical records (PDFs, faxes, structured EDI).
- External: NCCI loss costs, BLS wage data, provider directories, MPN data.

For most mid-size carriers, this data exists but is fragmented across the claims core, billing system, medical bill review vendor, and document repositories. Step zero is a **claims-360 data layer**.

**Aisha:** Endorsed. That's the data platform capability — practical: an event-driven ingestion from the core systems and vendors, normalized into a claims-centric data model, then served to the ML platform and operational dashboards.

**Priya:** Make sure the actuarial reserving group is a first-class consumer. If the platform produces a richer claims dataset, our reserve estimates get sharper and we shrink IBNR uncertainty.

---

## §6 — Regulatory guardrails (Linda)

**Linda:** Hard requirements going into the register:
- **R-COMP-01** — Full decision audit trail for every AI-influenced action.
- **R-COMP-02** — Bias testing pre-deployment + at least annual for every production model influencing claim outcomes.
- **R-COMP-03** — Human review for compensability, payment denial, and settlement above defined threshold.
- **R-COMP-04** — Jurisdiction-aware rules engine (no single-state assumption logic).
- **R-COMP-05** — Consumer-facing AI disclosure mechanism where required (CO, NY today; more coming).

**Daniel:** Add: **prompt pay** statutes. The platform cannot delay payments to chase efficiency. Penalty fines are not theoretical.

**Linda:** Right. **R-COMP-06** — Prompt pay enforcement: every state's statutory timelines coded into the rules engine and monitored.

---

## §7 — Build sequence (Ava drives)

**Ava:** Aisha — sequence the build. 36 months, target 2–2.5 LAE points.

**Aisha:** Three phases:

**Phase 1: Foundation (Months 0–9)**
- Claims-360 data platform (event ingestion + canonical model)
- Document AI for intake (medical records, FNOL artifacts)
- Severity scoring model v1 (FNOL features, simple GLM/GBM)
- Vendor performance scorecard (rules-driven first cut)
- Decision audit trail framework (cross-cutting)
- **Expected impact:** 0.4–0.6 LAE points (mostly ULAE — early severity routing, document handling)

**Phase 2: Optimization (Months 9–24)**
- Medical-only triage automation (with conservative STP routing)
- Real-time medical bill anomaly detection
- Litigation propensity model + early settlement workflow
- RTW orchestration
- Dynamic caseload balancing
- **Expected impact:** 1.0–1.2 LAE points (ALAE + ULAE)

**Phase 3: Closed Loop (Months 24–36)**
- Provider outcome scoring → network steering
- Vendor billing fraud detection
- Adjuster knowledge assistant
- Continuous model retraining infrastructure
- **Expected impact:** 0.6–0.7 LAE points

**Total expected impact:** 2.0–2.5 LAE points over 36 months — matches Rob's framing.

**Ava:** Realistic budget envelope, Aisha — rough order of magnitude?

**Aisha:** Build cost: $18–25M total over 36 months including integration, ML platform, change management. Run cost: $3–4M/year steady state. Payback inside year 3 on a $500M book.

**Priya:** I want to caveat that. The 2–2.5 point figure assumes execution discipline — change management, adjuster adoption, vendor renegotiation. Carriers that build the platform but don't change operating model capture maybe 40% of that.

**Sandy:** Endorsed. **The build is half the work. The other half is the operating model change.**

**Marcus (Scribe):** Capturing **Decision D-12: Program is co-equally a tech build and an operating model change. Funding plan must include change management headcount.**

---

## §8 — Open questions parked

**Ava:** Marcus, read out the open questions.

**Marcus:**
- **OQ-1** — Build vs partner for the document AI engine? (Owner: Aisha, decision by month 2)
- **OQ-2** — Which claims core system are we integrating with? Determines event integration patterns. (Owner: CIO)
- **OQ-3** — How aggressive is the executive team's appetite for vendor renegotiation? (Owner: COO)
- **OQ-4** — Will we adopt a unified data platform or extend the existing EDW? (Owner: Aisha + CDO)
- **OQ-5** — Adjuster compensation model — does it need to change to support outcome-based measurement? (Owner: Chief Claims Officer + HR)
- **OQ-6** — Which states first, for the jurisdiction-aware rules engine? (Owner: Rob + Linda — recommend top 5 states by premium)

---

## §9 — Closing

**Ava:** Marcus — finalize the requirements register and decisions log. Aisha and Rob co-own the requirements presentation to the executive sponsors. Aisha owns the developer briefing for the engineering team. Both delivered this week.

**Panel adjourned.**
