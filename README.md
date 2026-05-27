# WC LAE Optimization Platform — Working Set

Generated 2026-05-26 from an expert-panel session on building a platform to optimize the expense ratio (LAE-focused) for a mid-size monoline WC carrier ($300M–$1B DWP). Tech-agnostic deliverables.

## Contents

### `decks/` — final presentations
- **[`requirements_presentation.html`](decks/requirements_presentation.html)** — 16-slide animated HTML deck for the Executive Sponsor Group (CFO / COO / Chief Claims Officer / CIO). Business framing, target, capabilities, phasing, investment case, decision asked.
- **[`developer_briefing.html`](decks/developer_briefing.html)** — 17-slide animated HTML deck for the engineering team. System context, capability-domain deep-dives, contracts/events per domain, NFRs, integration surface, build sequence, Phase-1 definition of done.
- **[`functional_flow.html`](decks/functional_flow.html)** — 15-slide visual-first walkthrough of how the platform actually works end-to-end. Custom inline-SVG diagrams: platform overview, the 5-step claim journey, touchless vs lost-time scenarios, intake / bill / litigation / vendor flow details, data backbone, closed-loop learning, adjuster day before/after, AI-vs-human division of labor, KPI dashboards, one-picture summary. Designed for any audience.

Open any file directly in a browser. **Navigation:** `←` `→` arrow keys (or click left/right half of the screen). `F` for fullscreen.

### `panel/` — panel artifacts (source for the decks)
- **[`PANEL_TRANSCRIPT.md`](panel/PANEL_TRANSCRIPT.md)** — the working session: nine experts, nine sections, captured verbatim.
- **[`REQUIREMENTS_REGISTER.md`](panel/REQUIREMENTS_REGISTER.md)** — numbered functional + non-functional requirements derived from the panel.
- **[`DECISIONS_LOG.md`](panel/DECISIONS_LOG.md)** — 16 decisions taken in the panel, with rationale and dissent.
- **[`OPEN_QUESTIONS.md`](panel/OPEN_QUESTIONS.md)** — 8 questions parked with owners and due-dates.
- **[`GLOSSARY.md`](panel/GLOSSARY.md)** — plain-language definitions of every WC/regulatory/actuarial term used.

### `triage_decks/` — Session-II decks (Claims Triage & Vendor Match Layer)
Brand-new panel session that drills Domains 2 (Triage) + 5 (Vendor Orchestration) into one operational capability: a per-claim recommendation of *which services to engage* and *which vendor to engage them with*.
- **[`executive_brief.html`](triage_decks/executive_brief.html)** — 15-slide deck for the Executive Sponsor Group. The capability, the prize (0.8–1.2 LAE points · ~50% of the program target), the guardrails, the decision asked.
- **[`architecture_deck.html`](triage_decks/architecture_deck.html)** — 15-slide engineering deck. System context, four components, canonical data model, event catalog, NFRs, build sequence, open design questions.
- **[`match_flow.html`](triage_decks/match_flow.html)** — 15-slide visual walkthrough. One-picture overview, the 5-step recipe, three scenarios (routine · catastrophic · SIU red-flag), trigger map, scorecard visualized, adjuster screen mock, learning + attribution loops.

### `triage_panel/` — Session-II panel artifacts
- **[`PANEL_TRANSCRIPT.md`](triage_panel/PANEL_TRANSCRIPT.md)** — Session-II transcript: eight panelists, ten sections, captured verbatim.
- **[`DECISIONS_LOG.md`](triage_panel/DECISIONS_LOG.md)** — 18 decisions taken (D-T1 → D-T18) with rationale and dissent.
- **[`REQUIREMENTS_REGISTER.md`](triage_panel/REQUIREMENTS_REGISTER.md)** — functional + NFR requirements grouped by theme (catalog, triggers, pool, scoring, match, UX, learning loop, integration, compliance, risk).
- **[`SERVICE_CATALOG.md`](triage_panel/SERVICE_CATALOG.md)** — 36 services across 6 families with triggers, timing, unit-cost bands, dependencies, and a trigger→service quick map.
- **[`VENDOR_CATALOG.md`](triage_panel/VENDOR_CATALOG.md)** — 8-gate eligibility schema, 6-factor scoring schema with causal correction & decay, runtime match flow, and representative carrier-typical vendor pools per service.
- **[`OPEN_QUESTIONS.md`](triage_panel/OPEN_QUESTIONS.md)** — 10 items parked with owners and due dates.
- **[`GLOSSARY.md`](triage_panel/GLOSSARY.md)** — additions to the Session-I glossary (composite score, propensity weighting, exploration quota, holdout cohort, etc.).

### `agents/` — expert persona definitions
Nine specialized experts plus the meta-agents that ran the session:
1. [`01_moderator.md`](agents/01_moderator.md) — Dr. Ava Chen, Panel Moderator & Facilitator
2. [`02_scribe.md`](agents/02_scribe.md) — Marcus Reyes, Language Communicator & Note-Taker
3. [`03_wc_domain_expert.md`](agents/03_wc_domain_expert.md) — Rob Hutchinson, WC Domain Expert (32 yrs)
4. [`04_actuary.md`](agents/04_actuary.md) — Dr. Priya Sharma, FCAS — Pricing & Reserving
5. [`05_claims_ops.md`](agents/05_claims_ops.md) — Sandy Wexler, Claims Operations / LAE Specialist
6. [`06_underwriting.md`](agents/06_underwriting.md) — Daniel O'Brien, Underwriting & Acquisition Expense
7. [`07_tech_architect.md`](agents/07_tech_architect.md) — Aisha Patel, Insurance Technology Architect
8. [`08_data_science.md`](agents/08_data_science.md) — Dr. James Kim, Data Science & AI
9. [`09_regulatory.md`](agents/09_regulatory.md) — Linda Volkov, JD — Regulatory & Compliance

## Headline conclusions

| Item | Value |
|---|---|
| Target LAE reduction | 2.0–2.5 points over 36 months |
| Annual run-rate savings (on $500M DWP) | $10–12M |
| Build cost (36-month total) | $18–25M |
| Run cost (steady-state annual) | $3–4M |
| Payback | Inside year 3 |
| Posture | Layer above the claims core — not a replacement |
| Phasing | Foundation (M0–9) · Optimization (M9–24) · Closed Loop (M24–36) |

## The seven capability domains

1. Intake & Intelligence
2. Triage & Assignment
3. Medical Management
4. Litigation Management
5. Vendor Orchestration
6. Reserve & Reporting
7. Data & ML Platform

Plus cross-cutting: Audit · Explainability · Human-in-the-Loop.
