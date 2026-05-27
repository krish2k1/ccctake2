# Service Catalog — Claims Triage & Vendor Match Layer

36 services across 6 families. Each service: id, name, family, typical trigger, typical timing window from DOI (date of injury), typical unit-cost band (US national, 2026 baseline), and dependent / prerequisite services. Cost bands are indicative; jurisdictional and severity variance is material.

**Type column:** `DET` = deterministic rule, `PROB` = probabilistic classifier, `TIME` = timing-driven (decision is *when*, not *whether*), `HUMAN` = recommendation only, human approval required pre-engagement.

---

## Family A — Medical Direction & Care Coordination

| ID | Service | Type | Typical trigger | Timing window | Unit cost band | Depends on |
|---|---|---|---|---|---|---|
| A1 | Telephonic Nurse Case Management (TCM) | PROB + TIME | Lost-time flag; severity ≥0.4; opioid >14 days; treatment plan complexity | Day 3–14 | $1.5K–$5K per file lifetime | — |
| A2 | Field Nurse Case Management (FCM) | PROB + TIME | Severity ≥0.6 within 7 days; multi-provider treatment; surgery scheduled | Day 7–21 | $4K–$15K per file lifetime | A1 (often escalation) |
| A3 | Catastrophic Case Management | DET | Catastrophic flag (TBI, paralysis, amputation, ≥3rd-deg burn ≥30% TBSA, fatality, multi-trauma) | Within 24h of catastrophic flag | $15K–$80K+ per file lifetime | A2, C1 |
| A4 | Utilization Review (UR) — pre-certification | DET (most jurisdictions) | Treatment request for inpatient, surgical, imaging (jurisdiction-dependent) | Per treatment-request event | $150–$500 per review | — |
| A5 | Peer Review | PROB | UR escalation; treatment-plan no-improvement 60–90 days; medical-necessity dispute | Day 30+ as triggered | $400–$1.2K per review | A4 |
| A6 | Independent Medical Exam (IME) | PROB + TIME | Treatment plan no-improvement; disputed disability rating; pre-litigation evidence | Day 60–180 most common | $1.5K–$4K per exam | A5 often precedes |
| A7 | Functional Capacity Evaluation (FCE) | PROB | RTW assessment pre-release; disputed work capacity; MMI determination | Day 90+ | $800–$2K per evaluation | A6 often follows |
| A8 | Impairment Rating / MMI determination | DET (at MMI) | Treatment plateau reached; treating physician declares MMI | At MMI | $300–$1.2K | A7 |

## Family B — Medical Spend Management

| ID | Service | Type | Typical trigger | Timing window | Unit cost band | Depends on |
|---|---|---|---|---|---|---|
| B1 | Medical Bill Review (MBR) / Repricing | DET | Every medical bill received | Per bill | 18–28% of savings as fee, OR $5–$15 per line | — |
| B2 | PPO / MPN Network Steering | DET | Every provider referral / claimant request for care | Continuous | Network access fee + savings share | A1, A2 (NCM steers) |
| B3 | Pharmacy Benefit Management (PBM) | DET | First prescription on claim | Per prescription | Network discount + admin fee | — |
| B4 | DME (Durable Medical Equipment) | DET | DME prescription | Per item | Catalog pricing + delivery | A1/A2 coordinates |
| B5 | Diagnostic Imaging Network | DET | Imaging order (MRI, CT, advanced) | Per order | Network-rate per study | A4 often pre-certs |
| B6 | Physical Therapy Network | DET | PT prescription | Per episode | Per-visit network rate; episode caps | A4 pre-cert |
| B7 | Specialty Care (Pain Mgmt, Behavioral Health, PTSD) | PROB | Opioid escalation, chronic pain markers, mental-health-injury claim, traumatic event flag | Day 30+ | Specialty network rates; widely variable | A1, A5 |
| B8 | Hospital / Ambulatory Surgery Network | DET | Inpatient admission / surgery scheduled | Per event | Network-negotiated DRG/case rate | A4 |

## Family C — Return-to-Work & Indemnity Services

| ID | Service | Type | Typical trigger | Timing window | Unit cost band | Depends on |
|---|---|---|---|---|---|---|
| C1 | Vocational Rehabilitation | PROB | Permanent restriction probable; no return to prior job feasible; jurisdictional requirement | Day 60–180 | $3K–$15K per file | A3, A6, A7 |
| C2 | RTW Coordination Specialist | PROB | Lost-time + employer modified-duty unavailable / unclear | Day 7–30 | $1K–$4K per file | A1, C3 |
| C3 | Job Analysis / Ergonomic Assessment | PROB | RTW restrictions issued; modified-duty fit questioned | Day 14–60 | $500–$1.5K per analysis | A7 |
| C4 | Transitional Duty Placement / Modified-Duty Job Pool | PROB | Restrictions issued + no employer modified-duty | Day 14–45 | $1K–$3K per placement + wages | C2, C3 |
| C5 | Language Interpretation / Translation | DET | Claimant primary language ≠ English (per claimant data) | Every clinical / legal touch | $50–$150 per encounter | — |
| C6 | Transportation | DET | Claimant lacks transportation; medical appointment scheduled | Per appointment | $30–$120 per trip | — |

## Family D — Investigation & Defense

| ID | Service | Type | Typical trigger | Timing window | Unit cost band | Depends on |
|---|---|---|---|---|---|---|
| D1 | SIU / Fraud Investigation | PROB | Fraud-score ≥ threshold; lag-time anomaly; prior-claim pattern; provider on watch list | Day 7–30 | $1.5K–$8K per investigation | D3, D5 often parallel |
| D2 | Sub-rosa Surveillance | HUMAN (advisory) | Disability-vs-activity inconsistency signal; SIU recommendation | After human approval | $1.5K–$5K per assignment | D1 |
| D3 | AOE/COE Investigation | PROB | Compensability ambiguity at FNOL; lag-time; witness conflict | Day 1–14 | $800–$3K per investigation | — |
| D4 | Recorded Statement / EUO | PROB | Compensability ambiguity; SIU referral; major fact in dispute | Day 1–21 | $400–$1.5K per statement | D3 |
| D5 | ISO ClaimSearch / Background DB | DET | Every new claim above severity threshold | At FNOL | Per-query fee | — |
| D6 | Defense Counsel | PROB + TIME | Litigation propensity ≥0.5; attorney representation confirmed; subpoena received | Day 14–60 depending on signal | $250–$450/hr; $8K–$60K per matter | — |

## Family E — Resolution & Settlement

| ID | Service | Type | Typical trigger | Timing window | Unit cost band | Depends on |
|---|---|---|---|---|---|---|
| E1 | Mediation Services | PROB | Litigation present; settlement zone identified | Day 180+ typically | $1.5K–$5K per session | D6 |
| E2 | Structured Settlement Brokerage | PROB | Settlement contemplated > $100K; periodic-payment preference | Pre-settlement | Brokered commission | E3 often parallel |
| E3 | Medicare Set-Aside (MSA) Allocation | DET (Medicare eligible) | Settlement contemplated AND (claimant Medicare beneficiary OR future medical ≥ $25K with reasonable expectation of Medicare eligibility within 30 months) | Pre-settlement | $1.5K–$5K per allocation | A6, A8 |
| E4 | CMS Section 111 Reporting Agent | DET | Any settlement involving Medicare beneficiary | Per reportable event | Per-report fee | E3 |
| E5 | Medical Lien Resolution | PROB | Liens posted against settlement (Medicare, Medicaid, ERISA plan, provider) | Pre-settlement | % of lien reduction | E3 |
| E6 | Subrogation / Recovery Specialist | PROB | Third-party liability indicator (auto, premises, product) at FNOL | Day 30–90 | % of recovery | D3 |

## Family F — Records & Administrative

| ID | Service | Type | Typical trigger | Timing window | Unit cost band | Depends on |
|---|---|---|---|---|---|---|
| F1 | Medical Records Retrieval | DET | Records needed (any provider, any time) | Continuous | $0.10–$1.00 per page + base | — |
| F2 | Wage Statement / Employer Records | DET | Lost-time claim; AWW (average weekly wage) calculation needed | Day 1–14 | Per-request fee | — |
| F3 | EDI / State Regulatory Reporting | DET | Reportable event per state EDI schedule (FROI/SROI) | Per state schedule | Per-filing fee or platform fee | — |
| F4 | Document Translation | DET | Foreign-language document received; claimant non-English primary | Per document | $0.15–$0.40 per word | C5 |

---

## Service dependency graph (read top-to-bottom by typical claim progression)

```
FNOL
  ├─ F1, F2, F3 (always-on records / reporting)
  ├─ D5 (DB check — every above-threshold claim)
  ├─ D3 → D4 (compensability investigation if ambiguous)
  ├─ A4 (UR — per treatment request)
  │    └─ B1 (MBR — every bill)
  │         └─ B2 (steering — every provider)
  │              └─ B5/B6/B8 (modality networks)
  ├─ Severity ≥0.4
  │    └─ A1 (TCM)
  │         └─ A2 (FCM if escalates)
  │              └─ A3 (Cat NCM if catastrophic)
  │         └─ B3 (PBM — first Rx)
  │              └─ B7 (specialty care if opioid escalation)
  ├─ Lost-time
  │    ├─ C2 (RTW coordinator)
  │    │    └─ C3 (job analysis) → C4 (transitional duty)
  │    └─ C5/C6 (language / transport — always-on if needed)
  ├─ No-improvement at 60–90 days
  │    └─ A5 (peer review) → A6 (IME) → A7 (FCE) → A8 (impairment)
  │         └─ C1 (voc rehab if permanent restriction)
  ├─ Litigation propensity ≥0.5
  │    └─ D6 (counsel)
  │         └─ E1 (mediation)
  ├─ SIU triggers
  │    └─ D1 (SIU) → D2 (surveillance — HUMAN approval)
  ├─ Third-party liability
  │    └─ E6 (subrogation)
  └─ Settlement contemplated
       └─ E3 (MSA if Medicare exposure)
            └─ E4 (Section 111) + E5 (liens) + E2 (structured)
```

## Trigger → Service quick map

| Trigger | Services fired |
|---|---|
| Every new claim | F3, F1 (on first records), D5 (above threshold) |
| Lost-time (>7 days) | A1, C2, C3, C4 (sequenced) |
| Severity ≥ 0.6 within 7 days | A1, A2 |
| Catastrophic flag | A3, B8, C1 |
| Language ≠ English | C5, F4 |
| Opioid >14 days | A1 (if not already), B3 (intervention), B7 (specialty) |
| Lag-time >21 days | D1, D3, D4 |
| Litigation propensity ≥0.5 | D6, E1 (timing) |
| Treatment plateau (no improvement 60–90d) | A5, A6, A7 |
| MMI declared | A8, then C1 if restrictions permanent |
| Settlement contemplated + Medicare exposure | E3, E4, E5, E2 |
| Third-party liability indicator | E6 |
| SIU red flag | D1, parallel D3/D4; D2 only on human approval |
| Treating provider on watch list | A5, increased A4 scrutiny |
| Bill from out-of-network for in-network-available service | B1 (negotiation), B2 (re-steer) |
