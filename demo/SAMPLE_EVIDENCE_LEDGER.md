# Sample Evidence Ledger — INV-2026-0031

*Human-readable render of `fact_evidence_item` rows for INV-2026-0031, as a reviewer or committee member would see it drilling in from the committee report.*

| # | ID | Type | Source | Summary | Confidence | Counter-evidence? |
|---|---|---|---|---|---|---|
| 1 | EVID-2026-0031-01 | Policy clause | `dim_policy_clause` POL-MMP-3.2 | "A PSI value between 0.10 and 0.28 shall be classified Amber and shall trigger an investigation within 5 business days." | High | No |
| 2 | EVID-2026-0031-02 | Tool output | `get_segment_metric` (CSI decomposition) | Largest PSI contributors: utilization and bureau-derived feature groups | High | No |
| 3 | EVID-2026-0031-03 | Table row | `fact_application` (campaign filter) | 1,340 applications from CMP-2025-THINFILE-04, booked Dec 2025, reaching MOB 6 in June | High | No |
| 4 | EVID-2026-0031-04 | Table row | `fact_segment_monitoring_metric` sk=900112 | Thin-file segment share: 8.1% → 18.4% | High | No |
| 5 | EVID-2026-0031-05 | DQ check | `fact_data_quality_check` sk=331021, rule DQ-014 | Bureau missingness 0.42 vs. 0.11 baseline, 4,788 rows affected | High | No |
| 6 | EVID-2026-0031-06 | Table row | `fact_bureau_tradeline` (missingness sample) | Missingness concentrated 2026-06-03 to 2026-06-09, matches known outage window | High | No |
| 7 | EVID-2026-0031-07 | Policy clause | `dim_policy_clause` POL-COLL-1.4 | Referral threshold tightened 15→10 DPD, effective 2026-05-15 | Medium | No |
| 8 | EVID-2026-0031-08 | Tool output | `get_monitoring_metric` (bad rate, May cohort) | May mature-cohort bad rate stable at 3.1%, within 3.0–3.3% trailing range | High | **Yes — against deterioration hypothesis** |
| 9 | EVID-2026-0031-09 | Table row | `fact_segment_monitoring_metric` (adjusted cut) | PSI excluding campaign cohort and referral-affected DPD band: 0.091 (Green) | High | No |

## How to read this
Every row is independently verifiable — click through to the source table/row in the underlying warehouse or Dataverse instance. Item 8 is deliberately included even though it argues *against* the eventual conclusion for genuine deterioration: it is what a reviewer would look for to confirm the Investigation Agent actually tested an alternative explanation rather than settling on the first plausible story.

Full trace query and narrative: `scenarios/SCENARIO-001-psi-breach.md`.
