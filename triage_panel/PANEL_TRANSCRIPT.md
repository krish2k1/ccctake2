# Expert Panel Transcript — Session II
## Topic: A Claims Triage Layer that Identifies the Right Services and Matches the Best Vendor — per claim, per service, every time

**Date:** 2026-05-27
**Carrier profile anchor:** Monoline workers' comp, ~$300M–$1B DWP, multi-state (10–25 states)
**Primary lever this session:** ALAE leakage + early-intervention timing (the two biggest controllable cost drivers exposed in Session I)
**Stack:** Tech-agnostic. Capability-led.

**Moderator:** Dr. Ava Chen
**Scribe:** Marcus Reyes
**Panelists:** Rob Hutchinson (WC Domain), Dr. Priya Sharma (Actuarial), Sandy Wexler (Claims Ops), Daniel O'Brien (Underwriting), Aisha Patel (Tech Architect), Dr. James Kim (Data Science), Linda Volkov (Regulatory)

**Reference:** Session I produced the seven-domain LAE platform. This session drills two domains — **Triage & Assignment (D2)** and **Vendor Orchestration (D5)** — into a single operational capability: a **Claims Triage Service-Identification & Vendor-Match layer**. The deliverable is a per-claim recommendation of *which services to engage* and *which vendor to engage them with*.

---

## §1 — Re-anchor on the problem (Moderator)

**Ava:** New session, sharper question. Last week we proved LAE is the prize. Today we go after the two biggest leaks Sandy named: **vendor leakage (15–20% of waste)** and **late nurse case management (10–15%)**. Both are symptoms of the same underlying failure — *the carrier engages the wrong service at the wrong time with the wrong vendor*. We will design the layer that fixes that. Sandy, frame it.

**Sandy:** Here's the operating truth. On any given lost-time WC claim, an adjuster decides — usually by gut, often by habit — which of roughly thirty external services to engage and which vendor to send it to. There are no rate cards in the adjuster's head. There is no outcome history at the vendor level. And the timing is whenever the adjuster gets around to it, which on a 150-claim desk is "late."

**Rob:** That's exactly how the same back-strain claim gets nurse case management on day 60 at one desk and never on another. Adjuster discretion, no decision support, no learning loop.

**Ava:** So define the deliverable precisely. Marcus, log this.

**Marcus (Scribe):** Logging: the **Triage & Vendor Match Layer** = a capability that (a) inspects a claim's state, (b) identifies the **set of external services** that should be engaged right now, (c) selects the **best-fit vendor** for each service from a curated pool, (d) routes the recommendation to an adjuster who confirms or overrides, and (e) feeds the outcome back into vendor scoring.

**Ava:** Good. Two questions: how many service categories, and how many vendors per category. Sandy?

**Sandy:** In a mature WC operation, the catalog is roughly **30 service types** organized in 6 functional families. Mid-size carriers should be running **3–5 active vendors per category** — enough for competition, few enough to manage. Anything beyond 5 per category is leakage by definition.

**Priya:** Logged for the actuarial side: if we move spend from underperforming vendors to top-performers in each category — outcome-adjusted — we capture the entire 15–20% vendor leakage bucket and a meaningful slice of the timing bucket. That's roughly **0.8–1.2 LAE points** of the program's 2.0–2.5-point target. This layer alone is half the prize.

**Ava:** That's the headline number. The rest of this hour, we earn it.

---

## §2 — The service taxonomy (Rob + Sandy)

**Ava:** Rob, walk us through the service taxonomy. Not theory — what services does a mid-size monoline WC carrier actually buy?

**Rob:** Six families. I'll name them, Sandy fills in the operational triggers.

**Family A — Medical Direction & Care Coordination.** The clinical layer that steers treatment.
  - A1. Telephonic Nurse Case Management (TCM)
  - A2. Field Nurse Case Management (FCM) — in-person
  - A3. Catastrophic Case Management (Cat NCM) — paralysis, severe burns, TBI, amputation
  - A4. Utilization Review (UR) — pre-certification of treatment requests
  - A5. Peer Review — physician-to-physician medical-necessity review
  - A6. Independent Medical Examination (IME)
  - A7. Functional Capacity Evaluation (FCE)
  - A8. Impairment Rating / MMI determination

**Family B — Medical Spend Management.** What we pay for care.
  - B1. Medical Bill Review (MBR) / Repricing
  - B2. PPO / MPN Network Steering
  - B3. Pharmacy Benefit Management (PBM)
  - B4. Durable Medical Equipment (DME) Network
  - B5. Diagnostic Imaging / Radiology Network
  - B6. Physical Therapy Network
  - B7. Specialty Care — Pain Management, Behavioral Health, PTSD
  - B8. Hospital / Ambulatory Surgery Network

**Family C — Return-to-Work & Indemnity Services.** Getting the worker back.
  - C1. Vocational Rehabilitation
  - C2. RTW Coordination Specialist
  - C3. Job Analysis / Ergonomic Assessment
  - C4. Transitional Duty Placement / Modified-Duty Job Pool
  - C5. Language Interpretation / Translation
  - C6. Transportation

**Family D — Investigation & Defense.**
  - D1. SIU / Fraud Investigation
  - D2. Sub-rosa Surveillance
  - D3. AOE/COE (Arising-Out-Of / Course-Of-Employment) Investigation
  - D4. Recorded Statement / EUO
  - D5. ISO ClaimSearch / Background Database Checks
  - D6. Defense Counsel (jurisdiction-specific panel)

**Family E — Resolution & Settlement.**
  - E1. Mediation Services
  - E2. Structured Settlement Brokerage
  - E3. Medicare Set-Aside (MSA) Allocation
  - E4. CMS Section 111 Reporting Agent
  - E5. Medical Lien Resolution
  - E6. Subrogation / Recovery Specialist

**Family F — Records & Administrative.**
  - F1. Medical Records Retrieval
  - F2. Wage Statement / Employer Records
  - F3. EDI / State Regulatory Reporting
  - F4. Document Translation

**Rob:** That's 36 services. Most carriers fund all of them. Most carriers manage none of them well.

**Sandy:** And critically — **these are not independent.** A nurse referral often triggers PT network steering. An MSA review pre-supposes future medical estimation. UR feeds bill review. The layer must understand the **dependency graph**, not just a flat catalog.

**Marcus (Scribe):** Captured as the **Service Catalog**. Each service gets a card with: id, name, family, typical trigger, typical timing, typical unit cost, and dependencies on other services. Deliverable: `SERVICE_CATALOG.md`.

---

## §3 — What triggers each service (Sandy + James)

**Ava:** Now the hard part. What in the claim data tells us *this claim needs service X today*?

**Sandy:** I'll do the rule-based view, then James does the model view. A trigger is a claim attribute or event. Let me give the key ones, mapped to services:

| Trigger | Source | Services it should fire |
|---|---|---|
| Lost-time flag (>7 days disability) | Claim system | A1 (TCM), C3 (job analysis), C4 (modified duty) |
| Severity score ≥ 0.6 within 7 days of DOI | Severity model from Session I | A2 (FCM), early reserve review |
| Body part: spine, shoulder, knee + age ≥ 45 | FNOL + claimant data | A4 (UR for surgery preauth), B6 (PT) |
| Opioid prescription beyond 14 days | PBM feed | A1 (TCM), B3 (PBM intervention) |
| Litigation propensity score ≥ 0.5 | Litigation model | D6 (counsel), early settlement workflow |
| Lag time > 21 days between DOI and reporting | FNOL date − DOI | D1 (SIU triage), D3 (AOE/COE) |
| Surveillance red flags (prior similar claim, social media inconsistency) | Adjuster note NLP + ISO | D2 (surveillance), D1 (SIU) |
| Future medical estimate ≥ $25K with settlement contemplated | Reserve + settlement flag | E3 (MSA), E4 (Section 111) |
| Treating provider on watch list | Provider score | A5 (peer review), increased UR |
| Catastrophic indicators: TBI, paralysis, amputation, ≥3rd-deg burn, fatality | FNOL coded fields | A3 (Cat NCM), C1 (voc rehab), B8 (hospital steering) |
| Language indicator ≠ English | Claimant data | C5 (interpretation) at every clinical touch |
| Treatment plan no-improvement at 90 days | NCM notes + MBR | A6 (IME), A5 (peer review) |
| Bill from out-of-network provider for in-network-available service | Bill review feed | B1 (MBR negotiation), B2 (steering) |
| Distance from claimant to treating provider > 25 mi for routine care | Geo data | B2 (PPO steering closer in-network) |

**Rob:** That table is what every claims VP wishes they had on a wall. The trick is that most carriers have *the data to compute all of those* — it's just sprawled across the claims system, the bill review vendor's portal, the PBM portal, and adjuster note PDFs. The platform has to consolidate.

**James:** That's the model view. Some triggers are deterministic rules — language → interpreter, catastrophic → Cat NCM. No model needed; the cost of being wrong is too high. **Other triggers are probabilistic — when to send a field nurse, when to engage defense counsel, when to order surveillance.** Those are where models earn their pay.

**James (continuing):** I'd build the **service-identification layer as a hybrid**:
1. **Deterministic rules** for the unambiguous services (interpreter, MSA, Section 111, EDI). These are policy, not prediction.
2. **Multi-label classifier** for the discretionary services. Each service is a binary label; the model emits a probability per service plus a SHAP-style feature attribution per recommendation. Adjusters see *why*.
3. **Timing model** — for any service whose value depends on timing (FCM, defense counsel, surveillance), a survival-style model predicts the optimal engagement window. Late by 30 days = lost value.

**Priya:** And the timing piece is testable. We can A/B the "engage now vs engage in 2 weeks" recommendation on lower-severity claims and measure outcome differences. That's how we close the loop on timing leakage.

**Ava:** Linda — anywhere the deterministic-vs-probabilistic line is itself a regulatory question?

**Linda:** Yes, several. **Compensability denial cannot be an automated trigger** — but service engagement is not compensability denial, so the layer is safer than it sounds. However: anything that triggers *surveillance* on a specific claimant touches privacy and bias risk. We will need a documented surveillance-trigger policy with human approval before any sub-rosa activation. SIU has lower friction because it's investigation, not adverse action.

**Marcus (Scribe):** Logged: **R-TRIG-01** Deterministic vs probabilistic triggers are classified explicitly per service. **R-TRIG-02** Surveillance (D2) requires explicit human approval before vendor engagement, regardless of model recommendation.

---

## §4 — Curating the vendor pool (Sandy + Daniel + Linda)

**Ava:** We've identified services. Now — what makes a vendor *eligible* to be in the pool? Sandy.

**Sandy:** Eligibility is a binary gate. Performance is a continuous score. Don't confuse them.

**Eligibility gates** — a vendor either passes or doesn't:
1. **Licensure / credentials** — current and verified per state where they operate. Nurses, doctors, attorneys, investigators all have state-level credentials.
2. **Insurance** — E&O coverage at carrier-required limits.
3. **BAA executed** — HIPAA business associate agreement on file for anyone touching PHI.
4. **Compliance attestations** — NAIC AI bulletin where vendor uses AI, state surveillance laws for sub-rosa, MSA practitioner registration with CMS for MSA vendors.
5. **No active regulatory action** — DOI complaints, state bar discipline, CMS sanctions.
6. **Information-security posture** — SOC 2 Type II or equivalent for vendors transmitting claim data.
7. **Rate card filed** — every vendor must have a posted, machine-readable rate card. No rate card, no engagements.
8. **Capacity disclosed** — vendor publishes current intake capacity; we don't route into overflow.

**Daniel:** Add one from the underwriting/finance side: **conflict-of-interest disclosure**. A defense firm representing the carrier on Claim A should not be representing the same claimant's attorney on Claim B in the same jurisdiction. We need that asserted in the vendor master.

**Linda:** And per-state restrictions — some states bar certain vendor arrangements. California has specific MPN compliance. Texas has specific certified-network rules. The platform's vendor pool for each service must be **filtered by jurisdiction of the claim**.

**Ava:** OK — eligibility is the floor. Now the scoring layer that ranks the eligible. Sandy.

**Sandy:** Six factors. We weighted these from running scorecards at scale. The weights vary by service family but the structure is consistent.

| Factor | Weight (default) | Definition |
|---|---|---|
| **Outcome** | 40% | Outcome-adjusted result vs comparable claims handled by peer vendors. For NCM: outcome-adjusted RTW days. For MBR: net savings per file controlled for billed amount. For defense counsel: indemnity-plus-ALAE on closed files, controlled for venue and litigation propensity score at engagement. |
| **Cost** | 25% | Either savings-to-fee ratio (MBR, PBM, subrogation) or cost per file controlled for complexity. |
| **Speed / SLA** | 15% | Time-to-first-action, cycle time, SLA hit rate. |
| **Specialty fit** | 10% | Body part / injury type / language / geography / venue. Some vendors are excellent at upper-extremity but mediocre at psych. The score weights this per claim. |
| **Adjuster satisfaction** | 5% | Rolling qualitative score from adjuster post-engagement survey. |
| **Compliance / data quality** | 5% | Audit findings, complaint rate, EDI failure rate, late filings. |

**James:** Two technical refinements. First — outcome scoring needs **causal correction**. Top-performing vendors get the easiest claims because adjusters route to them. That looks like skill but is selection bias. We control for severity-at-engagement, litigation propensity at engagement, claimant demographics, jurisdiction, and venue. We use a **propensity-weighted estimator** so the outcome score reflects skill, not assignment.

**James (continuing):** Second — **time decay**. A vendor that was excellent two years ago may have lost staff. The score weights last 6 months at 60%, prior 7–18 months at 40%, drops anything older.

**Priya:** And we need a **minimum sample size** before a score is published. Below 30 closed engagements per service line per vendor, we publish a flag, not a rank. Otherwise we'll over-fit on noise and unfairly punish small vendors.

**Sandy:** Two operating guardrails I want on the record:
1. **Anti-concentration.** Don't route more than 50% of any service's volume to a single vendor, even if they score #1. Vendor capacity, vendor risk, and rate leverage all break with single-source.
2. **Anti-stale-pool.** Inject 5% of routing volume into the #4–#5 ranked vendors so the score has data to refresh. Cold vendors stay cold otherwise.

**Marcus (Scribe):** Captured: vendor scorecard schema, weights, decay function, sample-size floor, anti-concentration cap, exploration quota. Deliverable: `VENDOR_CATALOG.md` with schema and a sample of carrier-typical vendors per category.

---

## §5 — The match algorithm (James + Aisha)

**Ava:** Walk me through what happens at runtime, James. A claim event arrives. What does the layer do?

**James:** Five steps. Synchronous for the adjuster experience.

**Step 1 — Snapshot.** Pull the canonical claim state at this moment: FNOL data, latest severity, litigation propensity, body part / NCCI codes, jurisdiction, language, geography, current reserve, engaged services so far, treatment plan if known.

**Step 2 — Identify services.** Run the hybrid service-identifier:
  - Deterministic rules fire → required services list.
  - Multi-label classifier emits probability per discretionary service. Threshold per service (tuned per service ROI curve) decides which are recommended.
  - Timing model emits a "when" for any recommended service.

**Step 3 — For each recommended service, retrieve eligible vendor pool.** Apply hard filters: jurisdiction, credentials, capacity, conflicts, exclusion list, BAA. Eligible pool is typically 4–10 vendors per service per jurisdiction.

**Step 4 — Score and rank.** Each eligible vendor gets a composite score for *this* claim (not a generic score) — specialty-fit weights matter here. Top-3 emerge with rationale.

**Step 5 — Present.** Adjuster sees the recommendation in their workflow: services × top-3 vendors × rationale × estimated cost × estimated outcome impact. Adjuster confirms, swaps a vendor (with reason captured), or rejects (with reason captured). On confirm, the platform fires the vendor engagement event.

**Aisha:** Architecturally this is an orchestration loop, not a monolith. Three sub-services:
  - **Service Identifier** — service in the data/ML platform. Consumes claim snapshot, emits service list with confidence.
  - **Vendor Pool** — service backed by the vendor master + scoring data. Consumes service id + claim context, emits ranked candidates.
  - **Recommendation Composer** — composes both into an adjuster-facing recommendation with audit metadata. Persists every recommendation regardless of whether the adjuster acts on it.

**Aisha (continuing):** Three integration surfaces that matter:
1. **Inbound** — claim core events (FNOL filed, severity scored, treatment plan submitted, reserve changed, status changed). Event-driven so the layer recomputes when state changes.
2. **Outbound** — vendor engagement orchestration. EDI/API to the vendor, intake packet generation, capacity decrement, SLA timer started.
3. **Feedback** — outcome events from the vendor (cycle time, services rendered, invoices) and from the claim outcome (RTW date, closure, total cost). This is what trains the scorer.

**Ava:** Priya — any actuarial constraint on how often the recommendation is recomputed?

**Priya:** Yes. **Reserves and reasoning have to be reproducible at a point in time.** If the recommendation moves a reserve, we need the snapshot that drove it. Every recommendation persists with the snapshot inputs, the model versions, the weights, the score outputs. Auditable forever.

**Linda:** Co-sign. And add: **adjuster override reasons are not optional metadata**. They're regulatory artifacts. If a regulator later asks "why did you bypass the recommendation in 30% of severe back cases," we need to answer.

**Marcus (Scribe):** **R-MATCH-01** Every recommendation persisted with snapshot, model versions, weights, scores. **R-MATCH-02** Override reasons captured as a controlled vocabulary plus free text. **R-MATCH-03** Recomputation is event-triggered; recommendation is immutable once issued, with successor recommendations linked by lineage.

---

## §6 — How adjusters actually use it (Sandy + Rob)

**Ava:** Sandy — you've watched adjusters reject the algorithm before. How does this not get rejected?

**Sandy:** Three rules, hard-earned.

**Rule 1 — Don't take the decision. Improve the decision.** The recommendation never says "use this vendor." It says "based on 187 similar claims, these three vendors closed within 14% of each other on outcome, with these cost tradeoffs. Vendor A excels on speed; Vendor B on outcome; Vendor C is cheaper but has limited capacity this week." The adjuster picks.

**Rule 2 — Show your work.** Adjusters trust what they can audit. Every score factor visible, every comparable shown, every override pattern surfaced ("you've routed away from Vendor B four times this month — your colleagues haven't. Why?").

**Rule 3 — Make the override path frictionless.** Two-click max. Free-text reason allowed. But the reason gets *coded* by the platform and *trended*. We don't punish overrides — we learn from them.

**Rob:** And the format matters. Not a pop-up. Embedded in the workflow at the moment the adjuster is making the decision. The TCM referral screen has the top-3 NCM vendors right there. The defense-counsel-engagement screen has the top-3 firms. We design the layer to feel like *better defaults*, not *external authority*.

**Daniel:** From the operating side — measure the override rate per service per adjuster. High override = signal. Could mean the adjuster has local knowledge (good — capture it in features). Could mean the model is wrong in their book of business (good — retune). Could mean training problem (good — fix it). The override rate is a feature of the system, not a bug.

**Marcus (Scribe):** **R-UX-01** Recommendations embedded in adjuster workflow at the decision moment, not as a separate inbox. **R-UX-02** Top-3 + rationale + estimated impact for each option. **R-UX-03** Override coded + free-text + trended.

---

## §7 — The learning loop (James + Priya)

**Ava:** James — how does this not become a black box that drifts?

**James:** Four loops, running on different cadences.

**Loop 1 — Vendor score refresh.** Weekly. New closures feed the outcome model; scores update; rankings shift. Carriers and vendors get notified of material rank changes monthly.

**Loop 2 — Service identifier retrain.** Quarterly. New labeled claims (services engaged, outcomes observed) update the multi-label classifier. Holdout validation. Bias testing per the model risk policy. Documented in the model registry.

**Loop 3 — Trigger threshold calibration.** Monthly. For each service we evaluate the ROI curve at the current threshold — service-induced cost vs outcome impact. We tighten or loosen thresholds based on observed marginal value.

**Loop 4 — Override learning.** Monthly. Override patterns are reviewed by service. If overrides cluster on a specific feature (e.g., adjusters consistently swap recommended vendor B for vendor D on shoulder claims in Tampa), we test whether the model is missing a feature — typically a geographic or specialty signal — and update.

**Priya:** From the financial side, I want one more loop: **the savings attribution loop**. Every recommendation has a counterfactual estimate of what would have happened without it. Quarterly, we compare actual outcomes against the counterfactual and publish savings to finance. That's how we know the layer is earning its keep — and that's the number that makes the program defensible to the board.

**James:** That's a propensity-score-matched estimator. Treated = adjuster followed recommendation. Control = matched claims where they didn't or where the recommendation didn't exist (pre-rollout cohort). Doable, but requires we run a holdout cohort during early rollout. Cost: ~5% of volume held out for 6 months.

**Sandy:** I'll fight for that 5% holdout. Without it we'll spend three years arguing whether the layer actually worked.

---

## §8 — Build vs buy & the integration map (Aisha + Daniel)

**Ava:** Aisha — what do we build, what do we buy, what do we wrap?

**Aisha:** Three buckets.

**Buy:**
  - Document AI for vendor invoices, IME reports, medical records (existing — Session I).
  - Medical bill review engine (Mitchell, Coventry, Optum-class). Don't build a fee-schedule engine.
  - PBM platform if not already retained.
  - PPO/MPN credentialing data (existing networks: Coventry, Anthem, Aetna, regional WC networks).
  - Provider directories.
  - ISO ClaimSearch — buy access.

**Build:**
  - **Service Identifier** (rules + classifier + timing).
  - **Vendor Master + Scorecard** — the canonical vendor data store with rate cards, eligibility, scores. No vendor product covers this competently.
  - **Recommendation Composer** with audit and adjuster UX.
  - **Outcome attribution engine** — the propensity-matched estimator.
  - **Override capture + learning loop**.

**Wrap:**
  - Existing claims core for inbound events. Use the events we have; don't re-architect the core.
  - Existing bill review engine for outbound vendor invoicing.
  - Existing defense counsel panel system if any.

**Aisha (continuing):** Big integration surface to call out: **the vendor side of the API**. Vendors must accept structured intake packets, return structured outcomes, and post invoices in a standard form. We will need a Vendor API spec — and a degraded path (PDF intake, manual outcome entry) for vendors not yet on the API.

**Daniel:** Add the policy-system tie. If a vendor performs work on a claim that affects premium audit (e.g., a payroll reconciliation finding from an investigator), it has to land back in policy admin. Don't build the layer as an island.

**Ava:** Cost ROM, Aisha?

**Aisha:** Within the Session I envelope. The Triage & Vendor Match layer is largely **a rebudgeting of existing Domains 2 + 5 spend**, not an addition. Estimate **$4–6M of the $18–25M program** sits here, weighted to Phase 2 (Months 9–24). The Service Identifier is a Phase 1 build (rules engine + v1 classifier); full vendor scoring and the learning loop land in Phase 2.

---

## §9 — Risk register for this layer (Linda + Sandy + James)

**Ava:** Round-robin on what could break this. Linda first.

**Linda:**
  - **R-1 (Regulatory):** A state DOI inspects vendor selection logic and finds disparate impact in vendor steering by claimant demographic. Mitigation: bias testing on the recommendation outputs, not just on input models.
  - **R-2 (Regulatory):** Surveillance (D2) recommendation triggers a privacy claim. Mitigation: D2 requires explicit human approval; recommendation is for a human review, not an automatic engagement.
  - **R-3 (Regulatory):** Vendor steering interpreted as anti-competitive — favored vendor receives outsized share, smaller vendors challenge. Mitigation: published scoring methodology; exploration quota.

**Sandy:**
  - **R-4 (Adoption):** Adjusters refuse to use it because it feels like surveillance of *them*. Mitigation: opt-in pilot; visible value to the adjuster (less paperwork); no individual override-rate metric used in performance reviews until the layer is mature.
  - **R-5 (Vendor relations):** Top vendors learn the scoring system and game it. Mitigation: score on outcomes the vendor can't easily fabricate (independent claimant outcomes, regulator data, claimant surveys).
  - **R-6 (Operational):** A high-score vendor goes under or capacity-collapses; routing fails. Mitigation: anti-concentration cap (no >50% to one vendor); capacity ceilings; pre-qualified bench of #4–#5.

**James:**
  - **R-7 (Model):** Severity-at-engagement isn't fully observed, so causal correction is biased. Mitigation: hold out a randomized 5% cohort during rollout for true counterfactual.
  - **R-8 (Model):** Concept drift on injury patterns (e.g., remote-work claims, post-COVID long-haul, gig economy classification). Mitigation: drift monitor on input distribution; quarterly retrain.
  - **R-9 (Model):** Cold-start on new services or new vendors. Mitigation: published exploration quota; "Bayesian" prior derived from peer vendors until 30+ closed engagements accumulate.

**Daniel:**
  - **R-10 (Strategic):** Layer drives short-term cost reduction at the expense of claimant outcomes (e.g., aggressive PT denials trigger litigation). Mitigation: outcomes are the dominant scoring factor at 40%; cost is 25%. Watch DOI complaint ratios as a leading indicator.

**Marcus (Scribe):** Risk register logged; all carry forward as required mitigations in the requirements register.

---

## §10 — Decisions, sign-off, and what we hand over (Moderator)

**Ava:** Pulling it together. What are we deciding today?

Marcus reads back the proposed decisions; the panel marks each.

  - **D-T1.** The Triage & Vendor Match layer is a single coherent capability spanning Session-I Domains 2 and 5. *Unanimous.*
  - **D-T2.** The service catalog has **36 services in 6 families**. The catalog is governed; additions require operations + compliance review. *Unanimous.*
  - **D-T3.** Service identification is **hybrid: deterministic rules for unambiguous services + multi-label classifier for discretionary services + timing model for time-sensitive services.** *Unanimous.*
  - **D-T4.** Vendor eligibility is gated by **8 hard criteria** (licensure, insurance, BAA, compliance, no regulatory action, infosec, rate card, capacity). *Unanimous.*
  - **D-T5.** Vendor scoring uses **6 weighted factors** (Outcome 40 / Cost 25 / Speed 15 / Specialty fit 10 / Adjuster sat 5 / Compliance 5), with causal correction and 6/12 month time decay. *Unanimous.*
  - **D-T6.** Anti-concentration cap of **50% max share per vendor per service**. *Unanimous.*
  - **D-T7.** Exploration quota of **5% routing to ranks 4–5** to refresh data on cold vendors. *Unanimous.*
  - **D-T8.** Minimum sample of **30 closed engagements** before a vendor score is published as a rank (below that, a flag). *Unanimous.*
  - **D-T9.** Surveillance (D2) **requires explicit human approval** prior to vendor engagement; recommendation is advisory only. *Unanimous (Linda + Sandy emphatic).*
  - **D-T10.** Every recommendation is **persisted with snapshot inputs, model versions, weights, scores, and outcome** for audit. *Unanimous (Priya, Linda).*
  - **D-T11.** Override reasons are **controlled vocabulary + free text + trended**; not used for individual adjuster performance reviews in the first 12 months of rollout. *Unanimous (Sandy, Rob).*
  - **D-T12.** **5% randomized holdout cohort** for 6 months during initial rollout to enable counterfactual savings attribution. *Sandy + James for; Daniel notes the revenue tradeoff but accepts; carried.*
  - **D-T13.** Build / Buy: **Buy** the bill review, PBM, PPO data, ISO; **Build** the Service Identifier, Vendor Master+Scorecard, Recommendation Composer, Attribution engine. *Unanimous (Aisha).*
  - **D-T14.** Cost envelope: **$4–6M of the $18–25M program** sits in this layer. Phase-1 ships rules engine + v1 classifier + scorecard MVP. Phase-2 ships full scoring + learning loop. *Aisha; unanimous as ROM.*
  - **D-T15.** Outcome attribution published quarterly to finance using **propensity-matched estimator**. *Priya owns; unanimous.*
  - **D-T16.** Vendor scoring methodology is **published to vendors annually** with sufficient detail that they can self-improve, insufficient detail to game. *Sandy + Linda; unanimous.*

**Ava:** Sixteen decisions, clean. Open questions go on the parked list. Marcus, summarize what we hand over.

**Marcus (Scribe):** Five artifacts from this session:
  1. `PANEL_TRANSCRIPT.md` — this document.
  2. `DECISIONS_LOG.md` — the 16 decisions above with owners and rationale.
  3. `REQUIREMENTS_REGISTER.md` — functional + non-functional requirements derived from the discussion.
  4. `SERVICE_CATALOG.md` — the 36-service taxonomy with triggers, timing, dependencies.
  5. `VENDOR_CATALOG.md` — the eligibility schema, scoring model, and a representative carrier-typical vendor list per category.
  6. `OPEN_QUESTIONS.md` — items parked.
  7. `GLOSSARY.md` — additions to the Session-I glossary.

**Ava:** Adjourned. The decks team will translate this for the Sponsor Group, the architecture team, and the operating audience.
