# Glossary — Session II additions

Additions to the Session-I glossary; concepts new to or specific to the Triage & Vendor Match layer.

| Term | Plain-language definition | Why it matters here |
|---|---|---|
| **Service identification** | Deciding which external services to engage on a claim, at what time | The first half of this layer's job |
| **Vendor match** | Picking which vendor in a curated pool should perform a chosen service for a specific claim | The second half of this layer's job |
| **Eligibility gate** | A binary pass/fail criterion for whether a vendor may be considered at all | Keeps the pool clean — credential lapses, insurance lapses, BAA gaps remove vendors automatically |
| **Composite score** | A single 0–100 number that ranks an eligible vendor for a specific claim | What the adjuster sees; what the routing decision uses |
| **Specialty fit** | Per-claim weighting that captures match quality on body part, language, geography, venue | Why the #1 vendor for one claim may be the #4 vendor for another |
| **Propensity weighting** | Statistical technique that corrects for the fact that top vendors get easier claims (selection bias) | Without it, a vendor's "outcome" partly reflects assignment, not skill |
| **Time decay (vendor score)** | Weighting recent performance higher than older performance | A vendor that was great two years ago may have lost the team that made them great |
| **Sample-size floor** | Minimum number of closed engagements before a vendor receives a published rank (set at 30) | Below the floor, scoring is noise; we publish a flag instead |
| **Anti-concentration cap** | Max share of routed volume to any single vendor (set at 50%) per service per 90-day window | Prevents single-source dependency and rate-leverage loss |
| **Exploration quota** | Fixed share (5%) of routing intentionally sent to lower-ranked vendors to keep their scores fresh | Without exploration, cold vendors stay cold; the leaderboard freezes |
| **Recommendation lineage** | Linked chain of recommendations for the same claim across successive events | Auditable history when a recommendation changes as the claim evolves |
| **Override reason vocabulary** | Controlled list of categorized reasons an adjuster swaps or rejects a recommendation | Feeds the learning loop; satisfies regulator audit |
| **Counterfactual attribution** | Statistical estimate of what would have happened without the recommendation | The defensible savings number for finance and the board |
| **Holdout cohort** | Randomly selected slice of claims (5%) held out from the recommendation layer for a defined period | Makes counterfactual attribution credible — without it, savings claims are contested forever |
| **Catastrophic flag** | FNOL-coded indicator of catastrophic injury (TBI, paralysis, amputation, ≥3rd-deg burn ≥30% TBSA, fatality, multi-trauma) | Triggers immediate Cat NCM (A3) and bypasses normal routing |
| **AOE/COE** | Arising Out Of / Course Of employment — the compensability test for a WC injury | Drives whether D3 (AOE/COE investigation) fires |
| **MMI** | Maximum Medical Improvement — the point at which further treatment will not materially improve the condition | Triggers A8 (impairment rating), then C1 (voc rehab) if restrictions are permanent |
| **MSA** | Medicare Set-Aside — a sum carved out of a WC settlement to cover future Medicare-covered medical | Required when settling with Medicare-exposure beneficiaries; E3 service |
| **Section 111 (MMSEA)** | CMS reporting requirement for settlements involving Medicare beneficiaries | E4 service; statutory; failure carries penalties |
| **MPN / PPO** | Medical Provider Network / Preferred Provider Organization — the steerable medical network | B2 service; jurisdiction-specific rules (e.g., CA MPN, TX certified networks) |
| **EDI (FROI/SROI)** | Electronic Data Interchange — First Report of Injury / Subsequent Report of Injury filed with state regulators | F3 service; per-state schedules; non-negotiable |
| **TCM / FCM / Cat NCM** | Telephonic / Field / Catastrophic Nurse Case Management — escalating intensity of clinical coordination | A1 / A2 / A3 services; timing is the dominant ROI driver |
| **UR** | Utilization Review — pre-certification of medical treatment requests | A4 service; state-certified UROs required in many states |
| **IME** | Independent Medical Examination — a physician exam by a non-treating doctor | A6 service; used for medical-necessity disputes, MMI, impairment |
| **FCE** | Functional Capacity Evaluation — standardized assessment of work-capacity post-injury | A7 service; informs RTW and impairment |
| **Sub-rosa surveillance** | Covert observation of a claimant to test consistency between claimed disability and actual activity | D2 service; **human approval required before engagement** |
| **ISO ClaimSearch** | National claims database used to identify prior claims and patterns | D5 service; sole-source national resource |
| **AWW** | Average Weekly Wage — the basis for indemnity benefit calculation | F2 service input; jurisdictional formulas vary |
