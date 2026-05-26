# Decisions Log

| ID | Decision | Rationale | Owner | Dissent |
|---|---|---|---|---|
| D-1 | Program target = 2.0–2.5 LAE points reduction over 36 months | Aligns with achievable best-quartile gap on $300M–$1B DWP book | Ava | None |
| D-2 | Primary lever is LAE (ALAE + ULAE), not UW expense | Biggest controllable gap for monoline WC carriers | Priya, Rob | None |
| D-3 | Platform sits *above/around* claims core, not as a replacement | Mid-size carriers cannot afford or risk a core replacement | Aisha | None |
| D-4 | Seven capability domains as scope frame | Captures full LAE leverage chain | Aisha | None |
| D-5 | ML models embed inside capability domains, not as a separate AI module | Avoids "AI lab nobody uses" antipattern | James | None |
| D-6 | Compliance NFRs (audit, explainability, human-in-loop) are cross-cutting from day one | Cannot be retrofitted; regulatory exposure is real | Linda | None |
| D-7 | No auto-denial of claims; AI recommends, licensed adjuster decides | Statutory + reputational risk | Linda, Rob | None |
| D-8 | Touchless ceiling for med-only claims ~30–40%, conservatively routed | Realistic for WC; over-aggressive STP produces complaints | Sandy | Some pressure for higher target — declined |
| D-9 | Phase 1 (Months 0–9) Foundation: data layer + document AI + severity v1 + vendor scorecard + audit framework | Earliest payback levers; foundation for later phases | Aisha | None |
| D-10 | Phase 2 (Months 9–24) Optimization: triage automation, bill anomaly, litigation propensity, RTW, caseload balancing | Builds on data + model foundation | Aisha | None |
| D-11 | Phase 3 (Months 24–36) Closed Loop: provider scoring, fraud detection, GenAI assistant, retraining infra | Long-tail levers with feedback loops | Aisha | None |
| D-12 | Program is co-equally a tech build and an operating-model change; funding plan must include change-management headcount | Carriers capture ~40% of benefit without operating-model change | Sandy, Priya | None |
| D-13 | Adjuster knowledge assistant is GenAI; autonomous claim decisions are not | Bounded GenAI use; full autonomy not viable | James, Linda | None |
| D-14 | Reserve models stay explainable (no black-box) | Actuarial + DOI requirement | Priya, Linda | None |
| D-15 | Build cost envelope ROM $18–25M over 36 months; run cost $3–4M/year steady state | Aisha's experience-based estimate; refine in detailed design | Aisha | Treat as ROM only |
| D-16 | Top-5 states by premium prioritized for jurisdiction-aware rules engine pilot | Concentrates effort on highest-impact jurisdictions first | Rob, Linda | TBD which 5 (see OQ-6) |
