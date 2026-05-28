# Underwriting Workbench (UW WB) — Functional Specification

**Scope:** Monoline workers' compensation carrier · $300M–$1B DWP · multi-state (10–25 competitive-rated states).
**Audience:** Engineering, product, business analysts implementing or evaluating a UW workbench.
**What this is:** How the workbench works end-to-end, and what features it has. Functional only — no scoring models, no UX visuals, no infra prescriptions.

---

## 1. What it is

The **UW Workbench** is the underwriter's desktop. It is the system of work for every commercial WC submission from the moment a broker emails it until the policy is bound and handed off to policy admin — and then through every renewal cycle. It sits between the **producer-facing surface** (broker portal / submission email / MGA feed) and the **policy admin / rating / reinsurance / loss-control** systems behind it.

It does five jobs for the underwriter:

1. **Receive and clean** every submission into structured form.
2. **Decide whether to quote** it (clearance, appetite, authority).
3. **Enrich and analyze** it (loss runs, mods, class codes, public data).
4. **Price and present** the quote (rating, schedule credits/debits, subjectivities).
5. **Bind, issue, and queue for renewal.**

Everything else — referrals, loss control, audit, reinsurance, dashboards — supports those five jobs.

---

## 2. Who uses it

| Role | Why they're in it |
|---|---|
| **Underwriter** | Primary user. Owns the submission from triage to bind. |
| **Underwriting Assistant (UA)** | Cleans submissions, runs clearance, gathers data, builds the file. |
| **Underwriting Manager** | Handles referrals above an underwriter's authority. Reviews bind decisions. |
| **Chief Underwriting Officer** | Sees portfolio dashboards, appetite changes, exception trends. |
| **Producer (agent / broker / MGA)** | Submits business; checks submission status; receives quotes. |
| **Loss Control Consultant** | Receives pre-bind survey assignments; uploads safety reports. |
| **Premium Audit team** | Receives audit schedules at bind. |
| **Compliance** | Audits decisions; pulls reports for state DOI filings. |
| **Reinsurance team** | Sees facultative referrals; reviews large risks. |
| **Actuary** | Reads bind data for rate-adequacy monitoring. |

---

## 3. End-to-end flow (the lifecycle of a submission)

A submission moves through 14 stages. Each stage has entry conditions, work performed, and exit conditions. Most submissions flow forward; a meaningful share decline / withdraw / fail at each gate.

```
[1] INTAKE  →  [2] CLEARANCE  →  [3] APPETITE  →  [4] TRIAGE
    →  [5] ENRICHMENT  →  [6] RISK ANALYSIS  →  [7] RATING
    →  [8] SCHEDULE RATING  →  [9] AUTHORITY CHECK
    →  [10] LOSS CONTROL (if required)  →  [11] QUOTE
    →  [12] NEGOTIATION  →  [13] BIND  →  [14] ISSUE & HANDOFF
                                                       ↓
                                              [RENEWAL PIPELINE]
                                              (re-enters at [4])
```

### Stage 1 — Intake

**Entry:** A submission lands via one of four channels: producer email (ACORD 130 PDF + attachments), broker portal upload, MGA scheduled feed, or direct API.

**Work:**
- All inbound channels deposit submissions to a single intake queue.
- Document AI extracts structured fields from ACORD 130 and attachments:
  - Named insured, FEIN, addresses (mailing, all locations)
  - Effective date requested
  - Producer / broker / BOR letter present
  - Class codes (governing + secondary), payroll per class per location, headcount
  - Prior carrier(s), prior premiums, expiring policy number
  - 5-year loss runs (one per prior carrier)
  - Experience mod / interstate mod (if known)
  - Federal coverage indicators (USL&H, FELA, Jones Act, OCSLA, MELA)
  - Excluded officers / partners / sole proprietors
  - Safety program declarations
- Extracted fields are presented to the UA for verification. Missing fields are flagged.
- An "Acord 130 confidence score" reports how confident the extraction is per field.

**Exit:** A structured **Submission Record** exists with a unique submission_id, all attachments linked, extraction confidence noted, missing-data flags listed.

### Stage 2 — Clearance

**Entry:** Submission Record exists.

**Work:**
- The clearance engine searches existing submissions, in-force policies, declined risks, and prior renewals for matches against:
  - FEIN (primary key)
  - Legal name fuzzy match
  - DBA name fuzzy match
  - Location address(es) match
  - Principal/officer name match
- Possible outcomes:
  - **No match** → continue to appetite.
  - **In-force policy** → notify both producers; apply Broker of Record (BOR) rules; route to BOR workflow.
  - **Active submission with different producer** → BOR workflow; honor first-in-time per state rules.
  - **Declined within last 12 months** → present prior decline notes; require manager override to re-quote.
  - **Renewal in flight** → link to existing renewal, do not create new submission.

**Exit:** Submission either continues or routes to BOR / decline / merge.

### Stage 3 — Appetite

**Entry:** Cleared submission.

**Work:**
- The appetite engine evaluates the submission against the carrier's underwriting guide:
  - Class codes: in-appetite / restricted / prohibited
  - Geography: licensed states; restricted states; appetite for state
  - Size: payroll bands; per-account caps
  - Hazard profile: combined hazard tier per the guide
  - Operations: specific exclusions (e.g., asbestos, demolition, professional athletes)
  - Federal coverage exposure (USL&H requires specific reinsurance treatment)
  - Loss profile: presence of large losses; frequency outside tolerances
- Outcomes:
  - **Clear in-appetite** → continue.
  - **Conditional** → continue with a documented condition (e.g., requires loss control pre-bind).
  - **Out of appetite** → auto-decline with reason code; producer notified; can be referred to a specialty market if relationship exists.

**Exit:** Submission either advances or is declined with reason.

### Stage 4 — Triage to underwriter

**Entry:** In-appetite submission.

**Work:**
- The triage router assigns the submission to a specific underwriter based on:
  - **Producer assignment** (some producers have dedicated underwriters)
  - **Geography** (state-of-domicile rules)
  - **Class-code specialty** (construction risks routed to construction underwriters)
  - **Size band** (small accounts to small-account team; middle market to MM team)
  - **Current workload balancing** (capacity-aware)
  - **Renewal continuity** (renewals go back to the underwriter who wrote the prior term where possible)
- Assignment is logged with rationale; reassignment requires a manager action.
- The underwriter sees the new submission on their dashboard. SLA timer for first-touch starts.

**Exit:** Submission has an owner.

### Stage 5 — Data enrichment

**Entry:** Owned, in-appetite submission.

**Work:** The workbench automatically (and the UA manually for gaps) pulls data from external services into the file:

| Source | What we get |
|---|---|
| **NCCI Riskworkstation / data feed** | Experience mod, interstate mod, class-code combinations, large-deductible state filings |
| **Loss-run parser** | Structured claim history from each prior carrier's loss-run PDF |
| **NAIC / state DOI feeds** | Class-code rate filings, schedule-rating bands per state |
| **OSHA establishment search** | Citation history, fatalities, severity rate |
| **D&B / Cortera** | Business verification, years in business, financial signals |
| **LexisNexis (commercial)** | Principal background, litigation history, bankruptcy |
| **OFAC / SDN / Terrorism watchlists** | Sanctioned entity check |
| **State Secretary of State** | Legal entity status, registered agent |
| **BLS / NCCI benchmark payroll** | Payroll-per-employee plausibility check |
| **Workers' comp bureau (state-specific)** | State-specific filings; data calls; rating bureau experience |
| **MVR / fleet data** (where relevant) | For auto-related classes |
| **Public web / news** | Recent incidents, safety culture indicators |
| **Catastrophe data** (where relevant) | Earthquake / hurricane exposure for fixed-location risks |

For each enrichment, the source, retrieval time, payload, and any user overrides are logged.

**Exit:** Enriched file ready for risk analysis.

### Stage 6 — Risk analysis

**Entry:** Enriched submission.

**Work:**
- **Loss analysis:**
  - Frequency (claim count per $1M payroll, per year, by class)
  - Severity (average claim cost, large-loss flags)
  - Trend (3-year, 5-year direction)
  - Open vs closed mix
  - Loss-development factors applied to recent years
  - Comparison to expected losses for the class
- **Hazard analysis:**
  - Governing class hazard tier
  - Mix across multiple classes
  - Operations-specific hazards (heights, machinery, hazardous materials, driving)
  - Safety-program review (OSHA 300 logs, written safety program, training program, return-to-work program presence)
- **Account-level signals:**
  - Years in business
  - Recent ownership changes
  - Financial health signals
  - OSHA citation pattern
  - Geographic concentration
  - Employee turnover indicators
- The underwriter writes an analysis narrative; structured fields capture key conclusions.

**Exit:** Underwriter has formed a price opinion and any subjectivities to apply.

### Stage 7 — Rating

**Entry:** Risk analysis complete.

**Work:**
- The rating bridge calls the carrier's rating engine with:
  - Payroll by class by state by location
  - NCCI loss costs (current filed)
  - LCM (Loss Cost Multiplier) per state
  - Experience mod
  - Federal coverages (USL&H, etc.)
  - Minimum premium rules
  - Expense constants
  - Terrorism premium
  - Catastrophe loadings where filed
- The engine returns **manual premium** and **modified premium** (manual × mod).
- All inputs and outputs are persisted with the rating engine version and rate filing version.

**Exit:** Quotable base premium calculated.

### Stage 8 — Schedule rating

**Entry:** Base premium in hand.

**Work:**
- The workbench presents the state's filed schedule-rating bands (range of credits and debits available, typically ±25% to ±40%).
- The underwriter selects credits/debits per factor (e.g., +5% safety equipment, -10% management, +3% premises) with a justification note per factor.
- The workbench enforces:
  - Per-state max/min limits
  - Justification required for any non-zero factor
  - State-specific factor names and definitions
- Total schedule modification is computed; final premium calculated.

**Exit:** Final premium proposed.

### Stage 9 — Authority check

**Entry:** Final premium proposed.

**Work:**
- The authority matrix evaluates the proposed bind against the underwriter's authority along multiple axes:
  - **Premium size** (e.g., UW authority up to $250K; over → manager; over $1M → CUO; over $5M → board)
  - **Hazard tier**
  - **Schedule rating magnitude** (large credits/debits often need higher approval)
  - **Loss ratio acceptance** (poor loss runs may require manager sign-off regardless of size)
  - **Class-code restrictions** (some classes require specialty sign-off)
  - **Federal coverages** (USL&H always requires defined approval path)
  - **Multi-state policies** (each added state may need approval)
  - **Reinsurance attachment** (large limits → facultative referral)
- **Within authority** → underwriter can proceed.
- **Outside authority** → referral workflow with rationale, SLA, and approver routing.

**Exit:** Authority resolved (proceed or referral).

### Stage 10 — Loss control referral (conditional)

**Entry:** Submission flagged by appetite or underwriter as requiring pre-bind loss control.

**Work:**
- Loss control referral generated with scope (full survey, focused review, desk review).
- Routed to internal loss-control team or external vendor per geography.
- Survey output (safety findings, recommendations, photographs) attached to file.
- Recommendations classified as **pre-bind required**, **post-bind required**, or **advisory**.
- Pre-bind required items become subjectivities.

**Exit:** Loss control report attached to file with classified recommendations.

### Stage 11 — Quote

**Entry:** Authority cleared; any pre-bind loss control complete.

**Work:**
- The workbench generates the quote package:
  - **Proposal letter** (producer-facing)
  - **Premium summary** (manual / experience mod / schedule / final)
  - **Coverage summary** (states, federal coverages, limits, deductible if any)
  - **Conditions / subjectivities** (loss control commitments, payroll caps, signed safety addendum)
  - **Quote expiration date**
  - **Notice of consumer disclosures** as required per state
- Quote is delivered to producer via portal + email; receipt logged.

**Exit:** Quote in producer's hands; awaiting response.

### Stage 12 — Negotiation

**Entry:** Producer has questions, counter-offers, or requests changes.

**Work:**
- All counter-offer correspondence is captured against the submission record.
- Re-rating supported by amending inputs (additional states, revised payroll, additional class) and re-running stages 7–9.
- Each version is preserved (v1, v2, v3…) with diff and decision rationale.
- Adjuster, no, **underwriter** (this is UW WB) approves the final version.

**Exit:** Producer accepts a specific version, or quote expires / declines.

### Stage 13 — Bind

**Entry:** Producer accepts a quote version on behalf of the insured.

**Work:**
- Bind verification:
  - BOR letter (if needed) is on file
  - All subjectivities are met or recorded for post-bind compliance
  - Effective date is within bindable window
  - Producer is appointed in all states being written
  - Producer has binding authority for this size and class
  - Down payment / billing-plan confirmation present
- **Binder document** issued (interim coverage letter).
- Bound submission becomes a **policy-to-be-issued** record.

**Exit:** Coverage is bound; policy issuance queued.

### Stage 14 — Issue & handoff

**Entry:** Bound policy.

**Work:**
- Policy data sent to policy admin (Guidewire PolicyCenter / Duck Creek / in-house) via API.
- Premium-audit team scheduled per state and per-policy audit rules.
- Reinsurance team notified for cession reporting.
- Loss-control post-bind recommendations queued.
- Renewal anchor created (the renewal pipeline begins counting from inception).
- Submission stages closed; renewal pipeline opens.

**Exit:** Active policy in policy admin; UW WB shows the bound submission with read-only history.

### Renewal pipeline (continuous)

The workbench monitors every in-force policy and surfaces it for renewal:

| Days before expiration | Workbench action |
|---|---|
| **T-180** | Renewal eligibility pre-check: appetite still holds; class still in-appetite; account size still fits underwriter |
| **T-120** | Pre-renewal data refresh: latest loss runs requested; latest mod pulled; OSHA refreshed |
| **T-90** | Renewal submission created; routed to incumbent underwriter; SLA timer begins |
| **T-60** | Quote target — if not quoted by T-60, escalation |
| **T-45** | Producer notification if non-renewal contemplated (per state law) |
| **T-30** | Bind by deadline; binder issued |
| **T-0** | Policy inception; cycle restarts |

Non-renewal decisions follow state-specific notice rules and documented reason codes.

---

## 4. Feature catalog

A flat list of every feature in the workbench. Each has functional detail in §5.

| # | Feature | Section |
|---|---|---|
| F-01 | Submission Intake & Document AI | §5.1 |
| F-02 | Clearance Engine | §5.2 |
| F-03 | Appetite Engine | §5.3 |
| F-04 | Triage Router | §5.4 |
| F-05 | Submission File (the UW's main workspace) | §5.5 |
| F-06 | Data Enrichment Services | §5.6 |
| F-07 | Loss-Run Parser | §5.7 |
| F-08 | Experience Mod & Class-Code Service | §5.8 |
| F-09 | Risk Analysis Workbook | §5.9 |
| F-10 | Hazard Scorer | §5.10 |
| F-11 | Rating Bridge | §5.11 |
| F-12 | Schedule Rating Workbook | §5.12 |
| F-13 | Authority Matrix | §5.13 |
| F-14 | Referral Workflow | §5.14 |
| F-15 | Loss Control Referral | §5.15 |
| F-16 | Reinsurance Flagging | §5.16 |
| F-17 | Quote Generator | §5.17 |
| F-18 | Subjectivities Manager | §5.18 |
| F-19 | Negotiation / Version History | §5.19 |
| F-20 | Bind Workflow | §5.20 |
| F-21 | Issuance Handoff to Policy Admin | §5.21 |
| F-22 | Audit Scheduling Handoff | §5.22 |
| F-23 | Renewal Pipeline | §5.23 |
| F-24 | Producer Portal | §5.24 |
| F-25 | Producer Performance Scorecard | §5.25 |
| F-26 | UW Dashboards & Funnel Analytics | §5.26 |
| F-27 | Audit Trail & Compliance Pack | §5.27 |
| F-28 | Notification & SLA Engine | §5.28 |
| F-29 | Notes, Tasks & Calendar | §5.29 |
| F-30 | Reporting & Export | §5.30 |

---

## 5. Per-feature functional detail

### 5.1 Submission Intake & Document AI (F-01)

**Purpose:** Convert producer submissions of any form into a structured Submission Record.

**Inputs:** ACORD 130 PDFs, broker emails, MGA structured feeds, broker portal uploads, direct API submissions.

**Behavior:**
- Channel-specific listeners deposit raw documents to a unified intake queue.
- A document classifier identifies the document type (Acord 130, loss run, OSHA 300, safety program, BOR letter, supplemental questionnaire).
- For each typed document, the appropriate extraction model runs:
  - Acord 130: ~80 structured fields including the multi-row class-code/payroll/state grid
  - Loss runs: per-claim rows (date of loss, status, paid, reserve, cause)
  - Supplements: specific WC supplemental application fields
- Extraction confidence per field is reported; below-threshold fields are highlighted for UA verification.
- Email parsing extracts producer signature data and links the submission to the producer record.

**Outputs:** A Submission Record with structured fields, linked attachments, extraction confidence scores, and a missing-data checklist.

**Key automations:**
- Auto-link submissions to producer (BOR), to prior policy (renewal), and to prior submission (re-quote).
- Auto-create UA task for any field below confidence threshold.
- Auto-acknowledge to producer with submission_id and intake checklist.

### 5.2 Clearance Engine (F-02)

**Purpose:** Prevent duplicate work and detect BOR situations.

**Inputs:** Submission Record (FEIN, names, addresses, principals); existing submissions, in-force policies, declined submissions.

**Behavior:**
- Multi-key match: exact FEIN match is highest confidence; fuzzy name + address combinations supplement.
- A configurable scoring rule outputs a clearance verdict per match: NO_MATCH, POSSIBLE_MATCH, EXACT_MATCH.
- For matches against in-force policies or active submissions with a different producer, the BOR workflow is invoked.
- For matches against prior declines within a configurable window (e.g., 12 months), the submission is flagged with prior decline reasons; manager override required to proceed.
- For matches against an existing renewal cycle, the submission is auto-linked, not duplicated.

**Outputs:** Clearance verdict, matched record IDs, recommended next action (continue / BOR / re-quote / link to existing).

**Key automations:** Auto-merge of renewal-in-flight; auto-route of BOR situations with notification to both producers.

### 5.3 Appetite Engine (F-03)

**Purpose:** Apply the underwriting guide consistently and explainably.

**Inputs:** Submission Record + carrier's structured underwriting guide.

**Behavior:**
- The guide is data, not code. Each rule has: subject (class, state, size, hazard), operator, threshold, action (PASS, CONDITIONAL, DECLINE), reason code, and effective dates.
- The engine evaluates every rule against the submission and produces:
  - Overall verdict (CLEAR / CONDITIONAL / DECLINE)
  - Per-rule trace (which rules fired, with what data)
  - Aggregated conditions (e.g., "pre-bind loss control required because class hazard ≥ Tier 3")
- For DECLINE, the engine emits a producer-facing decline reason and an internal explanation.
- For CONDITIONAL, conditions become candidate subjectivities (§5.18).

**Outputs:** Appetite verdict + conditions + complete rule trace.

**Key automations:** Auto-decline with producer letter; auto-creation of pre-bind subjectivities for conditional approvals.

### 5.4 Triage Router (F-04)

**Purpose:** Send each submission to the right underwriter with the right SLA.

**Inputs:** Submission, producer-assignment table, underwriter capacity, underwriter specialties, renewal continuity flag.

**Behavior:**
- A weighted-rule router evaluates: producer dedication, geography, class specialty, size band, current capacity, renewal continuity, and language match (if relevant).
- Output is a primary assignee + a backup. Assignment is published as an event; appears on the UW dashboard.
- SLA timer starts: first-touch SLA (e.g., 2 business days), quote-by SLA (e.g., 7 business days).
- Underwriter or manager can reassign with reason code.

**Outputs:** Assignment record with rationale; SLA timers active.

### 5.5 Submission File (F-05)

**Purpose:** The underwriter's main workspace. Everything about a submission in one place.

**Tabs / sections inside the file:**

| Section | Contents |
|---|---|
| **Summary** | Insured, effective date, payroll, states, producer, current stage, SLA timers, key flags |
| **Documents** | All attachments with type tags, confidence scores, version history |
| **Coverage** | States, class codes & payrolls, federal coverages, limits, deductible, endorsements |
| **Loss History** | Parsed loss runs by year, large losses, trend charts, comparison to expected |
| **Experience Mod** | Current mod, prior mods, NCCI lookup details, mod-impacting losses |
| **Hazard** | Hazard tier per class, governing class call, account hazard score |
| **External data** | OSHA, D&B, Lexis, web findings — chronological |
| **Rating** | Inputs, rating engine response, manual premium, modified premium |
| **Schedule rating** | Per-factor credits/debits with justifications |
| **Subjectivities** | Pre-bind + post-bind conditions with status |
| **Loss control** | Survey report, recommendations, classifications, due dates |
| **Reinsurance** | Cession analysis, facultative referrals |
| **Authority** | Authority computation, referral status, approver(s), decisions |
| **Quote** | Proposal versions, delivery log, expiration |
| **Negotiation** | Counter-offer log, version diffs, decision rationale |
| **Bind** | Bind preconditions checklist, binder, billing plan |
| **Activity** | All actions, notes, system events with timestamps |
| **Audit pack** | One-click compliance pack export |

### 5.6 Data Enrichment Services (F-06)

**Purpose:** Pull external data automatically and consistently.

**Behavior:**
- A registry of enrichment providers; each registers its name, query inputs, response schema, freshness policy, and cost.
- The workbench fires a default enrichment bundle at the end of clearance; additional enrichments fire on triggers (e.g., principal-background pull when a new principal name appears).
- Each enrichment result is persisted with timestamp, query inputs, raw response, and parsed fields.
- Cached results respect freshness windows (e.g., NCCI mod 30 days; OSHA 90 days; D&B 180 days).
- Underwriter can force-refresh any source.
- Cost and frequency tracked per source per submission for ROI.

### 5.7 Loss-Run Parser (F-07)

**Purpose:** Turn PDF loss runs from N prior carriers into a unified structured loss history.

**Behavior:**
- Per-carrier templates handle variations in loss-run formats (top-20 carriers covered out of the box; rest are best-effort).
- Output schema:
  - Policy period
  - Claim number
  - Date of loss
  - Date of report (lag)
  - Class code at time of loss
  - State
  - Cause of loss (parsed)
  - Body part (parsed)
  - Status (open / closed)
  - Paid medical
  - Paid indemnity
  - Reserve medical
  - Reserve indemnity
  - Total incurred
  - Recoveries (subrogation, deductible reimbursement)
- Aggregations computed: by policy year, by class, by state, by cause.
- Loss-development factors applied to recent years using the carrier's selected triangles.
- Large losses (≥ configurable threshold) tagged for narrative review.
- Confidence scores per claim row; under-threshold rows queued for UA verification.

### 5.8 Experience Mod & Class-Code Service (F-08)

**Purpose:** Single source of truth for NCCI mods and class-code mechanics.

**Behavior:**
- Fetches current and prior interstate experience mod from NCCI; for states with independent rating bureaus (CA, NJ, NY, PA, TX, DE, IN, MA, MI, MN, NC, WI), fetches from each bureau.
- Reports the mod with: anniversary rating date, effective period, contributing policy years, mod-impacting individual claims.
- Maintains class-code metadata: code, description, governing-class candidacy, current loss cost per state per filing version.
- Validates class-code combinations: governing class call, secondary class permissibility per state, federal-coverage requirements.
- Provides class-code change history for the insured across submissions / policy terms.

### 5.9 Risk Analysis Workbook (F-09)

**Purpose:** Where the underwriter forms an opinion.

**Behavior:**
- Pre-populated views: loss trend chart, severity scatter, frequency vs benchmark, hazard mix, OSHA history, recent web findings.
- Narrative fields: account narrative, loss commentary, safety commentary, pricing rationale.
- Structured conclusions: account risk category (preferred / standard / substandard), pricing tier, rate adequacy commentary.
- The workbook is referenced from rating (§5.11), schedule rating (§5.12), and the quote letter (§5.17).

### 5.10 Hazard Scorer (F-10)

**Purpose:** Consistent hazard tiering across the book.

**Behavior:**
- Inputs: class-code mix and payroll weights; OSHA history; loss-cause profile; operations questionnaire answers; federal-coverage indicators.
- Outputs: a hazard tier (e.g., 1–5) per the carrier's filed/internal scale, with a per-factor breakdown.
- Hazard tier feeds: appetite engine, authority matrix, schedule-rating starting point.

### 5.11 Rating Bridge (F-11)

**Purpose:** Insulate the workbench from the rating engine's specifics; provide one calling pattern.

**Behavior:**
- Accepts a canonical rating request: payroll grid (class × state × location), mod, federal coverages, options (deductible, retro, dividend plan), and filing-version pin.
- Translates to the rating engine's native call format.
- Persists the request, response, and version pin (rating engine version + rate-filing version) on the submission file.
- Supports re-rating with input deltas; preserves rating history per quote version.
- Returns: manual premium per class per location; modified premium; expense constants; minimum premium; terrorism; cat loadings; total quotable base premium.

### 5.12 Schedule Rating Workbook (F-12)

**Purpose:** Apply state-filed schedule-rating credits/debits compliantly.

**Behavior:**
- Per state: the workbench loads the filed factor list (typically 5–10 factors covering management, employee selection, premises, equipment, classification peculiarities, safety devices) with their permissible ranges.
- The underwriter enters a value per factor with a free-text justification per non-zero value.
- The workbench enforces total maximum (capped per state) and per-factor maximum.
- Each entry is logged with timestamp and underwriter id.
- Output applied to base premium.

### 5.13 Authority Matrix (F-13)

**Purpose:** Tell the underwriter whether they can bind or must refer.

**Behavior:**
- A configurable matrix of authority limits per underwriter (or grade) per dimension:
  - Premium size
  - Hazard tier
  - Schedule rating magnitude
  - Loss-ratio acceptance
  - Class-code restrictions
  - Federal coverages
  - Number of states / multi-state complexity
  - Reinsurance facultative threshold
- The matrix evaluates the proposed bind and emits: WITHIN_AUTHORITY or REFERRAL_REQUIRED with the breaching dimension(s).
- For referrals, the matrix names the required approver(s) and any sequence (e.g., manager first, then CUO).

### 5.14 Referral Workflow (F-14)

**Purpose:** Route, track, and decision referrals fast.

**Behavior:**
- A referral record contains: requesting underwriter, target approver(s), reason, supporting analysis (a snapshot of the file), SLA.
- Approvers see referrals in their queue with priority (size, hazard, expiration date).
- Outcomes: APPROVED (proceed with terms as-is), APPROVED_WITH_CONDITIONS (with required subjectivity changes), DECLINED (rationale captured, returned to UW), DEFERRED (more info requested).
- All referral correspondence is tied to the submission file.

### 5.15 Loss Control Referral (F-15)

**Purpose:** Get pre-bind safety eyes on the right risks.

**Behavior:**
- Referral types: full survey, focused review, desk review.
- Routing: internal consultants (by geography and specialty) or external vendors per the loss-control vendor pool.
- The referral package includes: insured, locations, scope, hazard tier, prior survey history.
- Result documents (survey report, photographs, recommendations) flow back and attach to the submission file.
- Recommendations classified as PRE_BIND_REQUIRED (becomes a subjectivity), POST_BIND_REQUIRED (scheduled after bind), or ADVISORY (informational).

### 5.16 Reinsurance Flagging (F-16)

**Purpose:** Surface risks that exceed treaty terms.

**Behavior:**
- Treaty parameters maintained as data: per-occurrence retention, per-claimant retention, aggregate limits, excluded classes/states, federal-coverage facultative requirements.
- The workbench evaluates each submission against treaty terms; flags those needing facultative reinsurance.
- Facultative referrals routed to the reinsurance team with the analysis pack.

### 5.17 Quote Generator (F-17)

**Purpose:** Produce the producer-facing proposal in seconds, formatted to brand and compliant per state.

**Behavior:**
- Templates per state where state-specific disclosures vary.
- Inputs: rating output, schedule rating, subjectivities, coverage selections.
- Output documents:
  - Proposal letter
  - Premium summary
  - Coverage summary
  - Conditions and subjectivities
  - State-required notices
  - Quote expiration
- Delivery: portal post + email; both logged with delivery receipt.

### 5.18 Subjectivities Manager (F-18)

**Purpose:** Track conditions the insured must satisfy.

**Behavior:**
- Each subjectivity has: id, description, source (appetite / loss control / underwriter / referral), classification (pre-bind / post-bind), due date, evidence required, status.
- Pre-bind subjectivities block bind unless explicitly waived by an authorized approver (with rationale captured).
- Post-bind subjectivities create policy-level tasks that follow the policy through service.

### 5.19 Negotiation / Version History (F-19)

**Purpose:** Preserve the full negotiation record without losing earlier offers.

**Behavior:**
- Each material change creates a new quote version (v1, v2, v3…).
- Each version snapshots: rating inputs, premium output, schedule rating, subjectivities, coverage selections.
- A diff view shows what changed between versions.
- All producer correspondence is timestamped and attached.

### 5.20 Bind Workflow (F-20)

**Purpose:** Verify, log, and bind.

**Behavior:**
- A bind-preconditions checklist enforces:
  - Producer has authority for size, class, states
  - BOR letter on file if needed
  - Pre-bind subjectivities satisfied or waived
  - Effective date within bindable window
  - Down-payment / billing plan confirmed
  - Quote version selected matches the accepted offer
- Binder document issued; coverage effective per quote version.
- Bound submission becomes a policy-to-be-issued record.

### 5.21 Issuance Handoff to Policy Admin (F-21)

**Purpose:** Push policy data to the policy core cleanly.

**Behavior:**
- A canonical handoff payload (insured, coverage, class/payroll, mod, premium, endorsements, subjectivities, billing) is sent to the policy core via API.
- Acknowledgement received; policy number written back to the submission file.
- Post-issuance reconciliation: any policy-core changes (e.g., effective date shift) are reflected back on the UW WB record.

### 5.22 Audit Scheduling Handoff (F-22)

**Purpose:** Initiate premium audit at bind.

**Behavior:**
- At bind, the workbench computes the audit schedule per state and policy size (mail audit / phone audit / physical audit) and pushes to the premium audit system with the policy reference.
- Audit outcomes (final payroll, additional / return premium) flow back and feed actuarial and producer-performance datasets.

### 5.23 Renewal Pipeline (F-23)

**Purpose:** Make renewals as deliberate as new business.

**Behavior:**
- Continuous: every in-force policy has a renewal anchor; the workbench schedules T-180 / T-120 / T-90 / T-60 / T-45 / T-30 actions per the policy.
- T-90 generates a renewal submission, attaches the in-force policy data and prior decisions, refreshes loss runs and mod, and routes to the incumbent underwriter (continuity rule).
- Non-renewal decisions follow state-specific notice rules and documented reason codes; producer notifications generated automatically.

### 5.24 Producer Portal (F-24)

**Purpose:** Self-service for producers.

**Behavior:**
- Submit (ACORD 130 upload + supporting docs)
- See status (intake / clearance / appetite / quoted / bound / declined)
- See SLA countdowns
- View, accept, or counter quotes
- Manage BOR situations
- View renewal pipeline for their book
- Access appetite-letter library
- Access their own performance scorecard (§5.25)

### 5.25 Producer Performance Scorecard (F-25)

**Purpose:** Manage producer relationships with data.

**Behavior:**
- Per-producer metrics on a rolling 12-month window:
  - Submission count, hit ratio (quotes / submissions), bind ratio (binds / quotes)
  - Premium written, average premium
  - Loss ratio on bound business (with development)
  - Acord 130 quality (extraction confidence average)
  - Time-to-complete-submission (a quality signal)
  - Renewal retention
  - Cancellation and non-renewal rates
- Tier classification (e.g., Tier 1 strategic, Tier 2 active, Tier 3 transactional, Tier 4 on review).
- Tier drives appetite letters, dedicated-underwriter assignment, and SLA priority.

### 5.26 UW Dashboards & Funnel Analytics (F-26)

**Purpose:** Show what's in the funnel and where it leaks.

**Behavior:**
- Personal dashboard (per underwriter): owned submissions by stage, SLA breaches, today's calendar, today's tasks.
- Team dashboard (per manager): team submissions, capacity, SLA performance, referral queue depth.
- Book dashboard (per CUO): submission funnel (intake → cleared → in-appetite → quoted → bound), conversion at each stage, premium written vs plan, hazard mix, geographic mix, class mix, loss-ratio leading indicators.
- Producer dashboard (per producer-manager): top producers, declining producers, hit/bind ratios, loss ratios.

### 5.27 Audit Trail & Compliance Pack (F-27)

**Purpose:** Regulator-ready audit on demand.

**Behavior:**
- Every action on a submission file is persisted with actor, timestamp, before/after data, rationale (where required).
- One-click export of an audit pack per submission: timeline, all decisions and approvers, rating inputs/outputs with version pins, schedule-rating justifications, referral records, all external data pulls, all documents.
- DOI rate-filing trace: every quote can be traced to the exact filed rates and rules in effect at the time.
- NAIC AI Model Bulletin compliance: where ML influenced any decision (extraction, classification, scoring), the model version, inputs, and output are persisted.

### 5.28 Notification & SLA Engine (F-28)

**Purpose:** Keep work moving.

**Behavior:**
- SLA timers per stage with thresholds; warnings before breach; escalations after breach.
- Notifications via email, in-app, and (configurable) Slack/Teams for the underwriter, manager, and producer.
- Calendar integration (Outlook / Google) for first-touch reminders, bind deadlines, quote expirations, renewal triggers.

### 5.29 Notes, Tasks & Calendar (F-29)

**Purpose:** Working memory.

**Behavior:**
- Free-text notes per submission, with @-mentions to colleagues.
- Structured tasks (assignee, due date, status); appear on personal dashboard.
- Calendar view of all SLAs, due dates, and reminders across owned submissions.

### 5.30 Reporting & Export (F-30)

**Purpose:** Get data out for actuarial, finance, and regulator needs.

**Behavior:**
- Standard report library: bound-business by class/state/month, submission funnel, loss-ratio rollups, schedule-rating distribution, authority-exception trends, producer scorecards.
- Ad-hoc query: a sanctioned subset of submission and policy data available to analysts.
- DOI report generation: per-state required reports (e.g., experience reporting, large-deductible filings, market-conduct data) produced on schedule.
- Data export with PII controls and access logging.

---

## 6. Key data objects

The workbench is structured around these objects. Each has a stable ID and a full history.

| Object | One-line definition |
|---|---|
| **Submission** | A request from a producer for a quote on a specific insured and effective date. Owns the full lifecycle from intake to bind/decline. |
| **Insured** | The legal entity to be covered. Identified by FEIN (and Secretary-of-State / D&B records). |
| **Producer** | The agent, broker, or MGA who submitted. Has appointment, authority, performance. |
| **Quote Version** | A specific priced offer on a submission. Multiple versions per submission during negotiation. |
| **Subjectivity** | A condition on a quote — pre-bind blocking or post-bind tracked. |
| **Referral** | A request for higher-authority approval; has approvers, SLA, outcome. |
| **Loss Run** | A structured record of prior losses from one prior carrier. Multiple per submission. |
| **Mod** | An experience mod record (current + history) per insured per rating bureau. |
| **Enrichment Result** | One external data pull, persisted with query and raw response. |
| **Bind Record** | The decision and metadata at the moment of bind. |
| **Policy (UW WB view)** | A read-only reflection of the bound policy with its UW-time decisions intact. Drives renewal pipeline. |
| **Renewal Anchor** | The scheduling object that triggers renewal stages T-180 through T-0. |

---

## 7. Integration map

The workbench connects to many systems. Each connection has direction (in / out / both), cadence (sync / async / batch), and a stable contract.

| System | Direction | Cadence | Purpose |
|---|---|---|---|
| **Producer email gateway** | in | async | Receive submissions, BOR letters, supplements |
| **Producer portal** | both | sync | Self-service submit, status, accept quote |
| **MGA feed** | in | batch | Scheduled submission ingestion |
| **Rating engine** | both | sync | Manual / modified premium calculation |
| **Policy admin (PolicyCenter / Duck Creek / in-house)** | both | sync | Bound-policy handoff; in-force read-back; renewal anchors |
| **Premium audit system** | out | sync | Audit scheduling at bind; audit outcome read-back |
| **Reinsurance system** | both | sync | Treaty parameters; facultative referrals |
| **Loss-control system / vendors** | both | sync | Pre-bind survey assignment; report ingestion |
| **NCCI / state rating bureaus** | in | async | Experience mod, class codes, rate filings |
| **OSHA, D&B, LexisNexis, OFAC, BLS, SOS, news** | in | sync/async | Data enrichment |
| **State DOI portals** | out | batch | Required filings and reports |
| **CRM / producer management** | both | sync | Producer record, appointments, tiers |
| **Finance / accounting** | out | batch | Bound premium, audit premium |
| **Actuarial data warehouse** | out | batch | Bound business and decisions for monitoring |
| **Document store** | both | sync | Attachments and generated documents |
| **Email / calendar / Slack / Teams** | out | sync | Notifications |
| **Identity provider** | in | sync | SSO, role-based access |

---

## 8. What is in vs out of scope for "UW WB"

**In scope (the workbench owns):** submission lifecycle from intake through bind; the rules engine for clearance, appetite, and authority; rating inputs and schedule rating; quote and bind documents; renewal pipeline; producer-facing portal; UW dashboards and producer scorecards; UW audit trail.

**Out of scope (other systems own):** the rating engine itself (the workbench calls it); the policy administration system (the workbench hands off); claims (the LAE platform owns); premium audit execution (audit system owns); reinsurance treaty management (reinsurance system owns); general ledger / billing (finance owns); ML model training for extraction or scoring (the model platform owns; the workbench consumes the trained artifacts).

---

## 9. Cross-references

- LAE-platform program context: [`README.md`](../README.md) · [`panel/REQUIREMENTS_REGISTER.md`](../panel/REQUIREMENTS_REGISTER.md)
- Daniel O'Brien's underwriting perspective from Session I: [`agents/06_underwriting.md`](../agents/06_underwriting.md)
- This document is functional only — pricing models, ML approaches, and UX specs are explicitly out of scope here.
