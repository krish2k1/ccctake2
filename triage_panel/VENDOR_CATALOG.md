# Vendor Catalog — Eligibility, Scoring, and Carrier-Typical Vendor Pools

This document defines (1) the **eligibility schema** that admits a vendor into the pool, (2) the **scoring schema** that ranks eligible vendors per claim per service, and (3) **carrier-typical vendor pools** per service family. The vendor pools below are *representative of the WC vendor landscape* — actual carrier configuration depends on existing contracts, geographic footprint, and category strategy.

---

## 1. Eligibility schema (the 8 gates)

A vendor must clear every gate to enter the pool for a given service in a given jurisdiction. Failure of any gate removes the vendor from candidate selection until reinstated.

| # | Gate | Evidence artifact | Refresh cadence |
|---|---|---|---|
| 1 | State licensure / credentials current for the service offered in each jurisdiction routed | License IDs, state board look-ups, NPI for clinicians | Per state expiration; auto-checked monthly |
| 2 | E&O / professional liability insurance at carrier-required limits | Certificate of insurance | Annual + on-change |
| 3 | HIPAA Business Associate Agreement (BAA) executed | Signed BAA on file | Per contract renewal |
| 4 | Compliance attestations on file | NAIC AI bulletin attestation (where AI used), state surveillance-law attestation (D2), CMS MSA registration (E3), state EDI capability (F3) | Annual |
| 5 | No active regulatory action | DOI complaint check, state bar discipline (D6), CMS sanctions, OIG exclusion list | Quarterly |
| 6 | Information-security posture | SOC 2 Type II report or equivalent; pen-test summary; data-breach notification policy | Annual |
| 7 | Machine-readable rate card filed | Rate card per service per state, current effective dates | On contract / on amendment |
| 8 | Capacity disclosed | Weekly intake capacity per service per region | Weekly; recomputed at every routing decision |

**Per-claim filter** (applied after the 8 gates): jurisdiction of claim matches vendor licensed jurisdiction; no conflict of interest with claim parties; vendor not on the claim's explicit exclusion list.

---

## 2. Scoring schema

### 2.1 Composite score

```
Score(v, c, s) = 0.40 · Outcome(v, c, s)
               + 0.25 · Cost(v, c, s)
               + 0.15 · Speed(v, s)
               + 0.10 · SpecialtyFit(v, c, s)
               + 0.05 · AdjusterSat(v, s)
               + 0.05 · Compliance(v)
```

Where `v` = vendor, `c` = claim, `s` = service. All factors normalized 0–100. Weights are defaults; configurable per service family within governance approval.

### 2.2 Factor definitions

| Factor | Definition | Method |
|---|---|---|
| **Outcome** | Outcome-adjusted result vs peer vendors handling comparable claims. Service-specific outcome metric (RTW days for NCM, net savings for MBR, indemnity+ALAE for defense, etc.) | Propensity-score-weighted estimator. Controls: severity at engagement, litigation propensity at engagement, NCCI body part, jurisdiction, venue, claimant age band. |
| **Cost** | Either savings-to-fee ratio (MBR, PBM, subrogation, lien resolution) or cost-per-file controlled for complexity | Linear model controlling for complexity features. |
| **Speed / SLA** | Time-to-first-action + cycle time + SLA hit rate (composite) | Empirical from prior engagements. |
| **Specialty fit** | Per-claim match: body part, language, geography distance, venue history | Distance/match function; computed per claim, not stored as vendor attribute. |
| **Adjuster satisfaction** | Aggregate score from post-engagement adjuster surveys; published only after ≥10 responses | Survey instrument, rolling 12 months. |
| **Compliance** | DOI complaints, EDI failure rate, late filings, audit findings | Internal compliance dashboard + state data. |

### 2.3 Time decay

| Window | Weight |
|---|---|
| Most-recent 6 months | 60% |
| 7–18 months | 40% |
| > 18 months | excluded |

### 2.4 Sample-size floor

A vendor with < 30 closed engagements for a given service line is shown as **provisional (flagged)**, not as a published rank. Provisional vendors are eligible for exploration quota (see §3.3) but not for primary recommendation.

### 2.5 Governance overrides on the score

- **Anti-concentration cap (D-T6):** No single vendor may receive > 50% of routed volume per service per rolling 90-day window. The match algorithm enforces this by capping the top-ranked vendor's share and re-ranking when the cap is reached.
- **Exploration quota (D-T7):** 5% of routing volume is allocated to rank-4 and rank-5 vendors (or provisional vendors with sufficient eligibility evidence) to refresh cold-vendor data.
- **Surveillance carve-out (D-T9):** D2 (surveillance) recommendations never auto-route. Top-3 surveillance vendors are presented for named-human approval; routing follows that approval.

---

## 3. Match flow at runtime

```
1. SNAPSHOT       — pull canonical claim state
2. IDENTIFY       — services needed now (rules + classifier + timing)
3. FILTER         — apply 8 eligibility gates + per-claim filter, per service
4. SCORE          — composite score per eligible vendor per claim
5. RANK           — top-3 with rationale + factor breakdown
6. GOVERN         — apply anti-concentration cap, exploration quota
7. COMPOSE        — present to adjuster with cost/outcome bands
8. CAPTURE        — adjuster accept / swap / reject + reason code
9. ENGAGE         — vendor intake packet + SLA timer + capacity decrement
10. OBSERVE       — outcome events feed back into the scorer
```

P50 latency budget: < 2s end-to-end. P95: < 5s.

---

## 4. Carrier-typical vendor pools (representative)

The lists below are illustrative of the **national WC vendor landscape**. Carrier configuration depends on existing contracts and category strategy. For each category we recommend **3–5 active vendors** (D-T2 supporting guidance: above 5 = leakage).

> ⚠ The vendor names below are real organizations in the workers' comp vendor ecosystem, included for illustrative reference only. Inclusion is **not** an endorsement and does not reflect any carrier procurement decision. Score profiles are notional and used here only to demonstrate the schema.

### Family A — Medical Direction & Care Coordination

| Service | Representative vendors | Selection notes |
|---|---|---|
| A1 TCM | Genex, Paradigm, CorVel, Concentra, Mitchell ScriptAdvisor (clinical arm) | Pool by region; field-vs-telephonic specialty matters |
| A2 FCM | Paradigm (catastrophic), Genex, CorVel, Concentra, regional independents | Geography & specialty fit weighted heavily |
| A3 Cat NCM | Paradigm Catastrophic, Genex Catastrophic, ISYS, Med-Cor Health Systems | Sub-specialty (TBI / spinal cord / burn / amputation) matters; small pool by design |
| A4 UR | Conduent, Coventry, Optum WC, Rising Medical Solutions | High-volume; jurisdictional certification critical (CA URO, NY workers' comp panel, etc.) |
| A5 Peer Review | MES Solutions, ExamWorks, Dane Street | Specialty match (orthopedic, neurology, psychiatry) drives selection |
| A6 IME | ExamWorks, MES Solutions, Dane Street, MLS Group | Specialty + venue + scheduling capacity |
| A7 FCE | Industrial Rehabilitation Group, local PT-network FCE providers | Geographic proximity to claimant |
| A8 Impairment Rating | ExamWorks, treating MDs with rating certification | Often part of A6 engagement |

### Family B — Medical Spend Management

| Service | Representative vendors | Selection notes |
|---|---|---|
| B1 MBR | Mitchell SmartAdvisor, Coventry First Health, Optum WC, Conduent, Rising Medical, Carisk | Savings-to-fee ratio is the dominant factor; rules engine quality matters |
| B2 PPO/MPN | Coventry, Anthem WC, Aetna WC, First Health, Prime Health Services, Rockport Healthcare | Network depth + discount + steerability per geography |
| B3 PBM | myMatrixx (Evernorth), Optum WC PBM, Mitchell ScriptAdvisor, Carisk Rx | Clinical opioid management + formulary discipline |
| B4 DME | One Call, Homelink, Apria, regional DME networks | Service speed + rental-vs-buy discipline |
| B5 Imaging | One Call (radiology network), MedRisk Imaging, Align Networks | Network rate vs. local availability |
| B6 PT | MedRisk, One Call PT, HealthPro Rehab, Align Networks | Episode caps + outcome metrics |
| B7 Specialty Care | IntegraNet (behavioral), local specialty providers, Rising Medical specialty network | Per-specialty pool; often regional |
| B8 Hospital/ASC | Direct contracts + national hospital networks (Coventry, First Health) | Surgery cost is the largest single line item — separate scoring weight |

### Family C — RTW & Indemnity Services

| Service | Representative vendors | Selection notes |
|---|---|---|
| C1 Vocational Rehab | International Vocational Rehabilitation Solutions, OccuVoc, Genex Voc, regional state-certified providers | State-certification critical (CA QRR, FL approved voc) |
| C2 RTW Coordination | Coventry Return-to-Work, Genex RTW, ReEmployAbility | Modified-duty placement track record is key |
| C3 Job Analysis | Same as C2 or specialist ergonomists | Often bundled with C2 |
| C4 Transitional Duty Placement | ReEmployAbility, MedAllies, employer-direct programs | Placement rate + claimant satisfaction |
| C5 Interpretation | Language Line, LanguageLine Solutions, Cyracom, regional in-person interpreters | Language coverage + medical-grade certification |
| C6 Transportation | One Call Transportation, Mod-Trans, regional NEMT providers | Geographic coverage + claimant satisfaction |

### Family D — Investigation & Defense

| Service | Representative vendors | Selection notes |
|---|---|---|
| D1 SIU | Frasco, AISG, Marshall Investigative Group, ICS | Outcomes vs spend; fraud-confirmation rate |
| D2 Surveillance | Frasco, AISG, Marshall, ICS (same pool, separate engagements) | **Human approval required.** State-law compliance critical |
| D3 AOE/COE | Frasco, AISG, regional investigators | Same pool as D1 typically |
| D4 Recorded Statement | In-house adjusters often; vendors for complex / multi-language | C5 may co-engage |
| D5 ISO ClaimSearch | Verisk / ISO ClaimSearch — sole-source national | National DB; no real alternative |
| D6 Defense Counsel | Carrier-built panel: regional WC defense firms per state. National coordinators (Marshall Dennehey, Lewis Brisbois, Wilson Elser) for multi-state programs | **By jurisdiction; venue history weighted heavily.** Conflict-of-interest checks per case |

### Family E — Resolution & Settlement

| Service | Representative vendors | Selection notes |
|---|---|---|
| E1 Mediation | JAMS, AAA panels, regional WC mediators | Venue + mediator track record |
| E2 Structured Settlement | Ringler Associates, Arcadia Settlements, Atlas Settlement Group, NFP Structured Settlements | Annuity-market access + claimant-rep skill |
| E3 MSA Allocation | ISO MSA, Tower MSA Partners, Ametros, ExamWorks Clinical Solutions | CMS submission success rate + clinical defensibility |
| E4 Section 111 Reporting | Same as E3 or specialized agents | Often bundled with E3 |
| E5 Lien Resolution | Garretson Resolution Group, Synergy Settlement Services, Precision Resolution | Reduction percentage achieved |
| E6 Subrogation | The Rawlings Group, Equian (Optum), MedRecovery Management, in-house teams | Recovery rate net of fee |

### Family F — Records & Administrative

| Service | Representative vendors | Selection notes |
|---|---|---|
| F1 Medical Records Retrieval | Verisma, ScanSTAT, MRO, ChartSwap | Cycle time + completeness |
| F2 Wage Statement Retrieval | Often handled in-house or via employer portal; specialist vendors include The Hartford Wage Verification Services and regional providers | Employer-relationship critical |
| F3 EDI / State Reporting | Verisk-IAIABC, ISO EDI, Mitchell Decision Point | State-by-state certification |
| F4 Document Translation | LanguageLine, Cyracom, Lionbridge, Stratus | Volume + medical/legal specialization |

---

## 5. Sample vendor scorecard (notional, illustrative format)

| Vendor | Service | Jurisdiction | Sample (N closed) | Outcome | Cost | Speed | SpecialtyFit (this claim) | AdjSat | Compl. | Composite | Rank | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Genex | A1 TCM | TX | 412 | 81 | 76 | 88 | 92 (shoulder) | 78 | 95 | **82.6** | 1 | Strong shoulder TCM record TX |
| Paradigm | A1 TCM | TX | 287 | 84 | 70 | 81 | 84 (shoulder) | 81 | 95 | **81.3** | 2 | Higher outcome, higher cost |
| CorVel | A1 TCM | TX | 356 | 77 | 82 | 84 | 78 (shoulder) | 75 | 92 | **79.4** | 3 | Cost leader |
| Concentra | A1 TCM | TX | 198 | 74 | 79 | 80 | 76 (shoulder) | 72 | 90 | **76.7** | 4 | Exploration-quota eligible |
| Regional Indep. X | A1 TCM | TX | 24 | — | — | — | — | — | — | **provisional** | — | < 30 sample; flagged not ranked |

Note: this is a per-claim scorecard (specialty fit is computed for this specific shoulder claim in Texas). For a different claim the rank may change materially.
