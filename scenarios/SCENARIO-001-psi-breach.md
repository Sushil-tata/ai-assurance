# Scenario 001 — Credit Card Behaviour Score PSI Amber, June 2026

**Status:** MVP primary demo scenario. Every ID below is used consistently across `data/TABLE_CATALOG.md` example rows, `demo/SAMPLE_EVIDENCE_LEDGER.md`, and `demo/SAMPLE_COMMITTEE_REPORT.md` — this file is the source of truth for the narrative; the others render it in their respective formats.

## 1. The engineered situation

Three independent causes are deliberately layered into the synthetic June 2026 data so the investigation has to disentangle them, not just detect one obvious breach:

1. **Population mix shift.** A digital acquisition campaign (`CMP-2025-THINFILE-04`) targeted thin-file customers (fewer than 2 active bureau tradelines) from December 2025. Those accounts reach MOB 6 in June 2026, entering the Behaviour Score scoring population for the first time and shifting its composition.
2. **Bureau feature missingness.** An upstream bureau feed (`BRU-FEED-07`) had a partial processing fault in June, causing `is_missing_flag = true` on more tradelines than normal for a subset of customers, raising `bureau_missingness_pct` and pushing several bureau-derived features toward their imputed/default values.
3. **Policy strategy change.** Collections Strategy tightened the early-collections referral rule on 2026-05-15, changing which accounts remain in the "current" scored population versus getting excluded pending collections referral — a scoped-population change, not a customer-behaviour change.

## 2. Trigger

`fact_monitoring_metric` row `monitoring_metric_sk = 810234`: `model_version_id = CC-BSCORE-v2.1`, `metric_code = PSI`, `calculation_run_id = RUN-MON-2026-06`, `run_date = 2026-07-05`, `metric_value = 0.263`, `rag_status = amber` (threshold `THR-CC-B-01`: green ≤ 0.10, amber ≤ 0.25 — 0.263 is technically just past the amber ceiling into what a naive read would call red; the actual threshold row in effect from `2026-04-01` widened the amber band to 0.28 following a prior committee-approved seasonal adjustment, so `rag_status = amber` is correct and itself becomes an evidence item establishing which threshold version applied).

The Monitoring Agent detects this on its scheduled run and opens:

`fact_investigation`: `investigation_id = INV-2026-0031`, `model_version_id = CC-BSCORE-v2.1`, `trigger_type = metric_breach`, `trigger_metric_sk = 810234`, `trigger_description = "PSI amber (0.263) for CC Behaviour Score, June 2026 run"`, `opened_date = 2026-07-05`, `opened_by_agent_id = AGT-MONITORING`, `severity = medium`, `status = open`.

## 3. Hypotheses (Investigation Agent, not persisted until evidence-tested)

- **H1:** Population mix shift from campaign-driven thin-file cohort reaching MOB 6.
- **H2:** Bureau feature missingness from an upstream data-quality fault.
- **H3:** Policy-driven change in the scored population definition.
- **H4 (tested and rejected):** Genuine behavioural/economic deterioration in the customer base. Tested via bad-rate trend on the mature May cohort — found stable, no supporting evidence, logged as a rejected hypothesis.

## 4. Evidence gathered (each a `fact_evidence_item` row, `investigation_id = INV-2026-0031`)

| `evidence_item_id` | Type | Source | Summary | Supports | Confidence | Contradictory? |
|---|---|---|---|---|---|---|
| `EVID-2026-0031-01` | policy_clause | `dim_policy_clause` `POL-MMP-3.2` | "A PSI value between 0.10 and 0.28 (per threshold version effective 2026-04-01) shall be classified Amber and shall trigger an investigation within 5 business days." | Establishes the correct threshold basis for the trigger itself | High | No |
| `EVID-2026-0031-02` | tool_output | `fact_segment_monitoring_metric` via `get_segment_metric` tool | CSI decomposition shows the largest single contributor to score-level PSI is the `avg_utilization_3m` and bureau-derived feature group, not a uniform shift across all features | H1 and H2 (narrows which features moved) | High | No |
| `EVID-2026-0031-03` | table_row | `fact_application` filtered by `campaign_id = CMP-2025-THINFILE-04` | 1,340 applications from this campaign booked Dec 2025, now reaching MOB 6 in June 2026 | H1 | High | No |
| `EVID-2026-0031-04` | table_row | `fact_segment_monitoring_metric` `segment_monitoring_metric_sk=900112` | Thin-file segment share of scored population rose from 8.1% (May) to 18.4% (June) | H1 | High | No |
| `EVID-2026-0031-05` | dq_check | `fact_data_quality_check` `dq_check_sk=331021`, rule `DQ-014` | Bureau aggregate missingness check **failed**: `bureau_missingness_pct` for June averaged 0.42 vs. a 0.11 rolling baseline, concentrated in accounts sourced from `BRU-FEED-07` | H2 | High | No |
| `EVID-2026-0031-06` | table_row | `fact_bureau_tradeline` sample, `is_missing_flag = true` rows | Missingness concentrated in tradelines pulled `2026-06-03` through `2026-06-09`, consistent with the feed's known outage window | H2 | High | No |
| `EVID-2026-0031-07` | policy_clause | `dim_policy_clause` (Collections Strategy referral rule, `POL-COLL-1.4`) | Early-collections referral threshold tightened from 15 DPD to 10 DPD effective 2026-05-15, removing a cohort of 5–14 DPD accounts from the "current, scorable" population starting late May | H3 | Medium | No |
| `EVID-2026-0031-08` | tool_output | `get_monitoring_metric` on `bad_rate`, May mature cohort | May mature-cohort bad rate stable at 3.1%, consistent with the 3-month trailing average (3.0–3.3%) | **Counter-evidence to H4** | High | **Yes** |
| `EVID-2026-0031-09` | table_row | `fact_segment_monitoring_metric`, non-campaign segment PSI cut | PSI computed *excluding* the campaign cohort and *excluding* the referral-rule-affected DPD band falls to 0.091 (Green) | Directly quantifies H1+H3's combined contribution — the residual, adjusted PSI is well within tolerance | High | No |

Note the structure: `EVID-2026-0031-08` is explicitly marked `is_contradictory = true` against H4 (deterioration), which is why H4 was rejected rather than simply not mentioned — the Assurance Agent requires evidence that a plausible alternative was actively tested, not just ignored.

## 5. Finding

`fact_finding`: `finding_id = FIND-2026-0031-01`, `investigation_id = INV-2026-0031`, `finding_statement = "The June PSI Amber breach is primarily attributable to a mix shift toward thin-file customers following campaign CMP-2025-THINFILE-04, compounded by bureau feature missingness from an upstream feed fault, with a secondary contribution from a Collections Strategy referral-rule change that altered the scored population definition. Excluding these three known population-composition effects, residual PSI is 0.091 (Green). No evidence of genuine behavioural or credit-quality deterioration was found; the May mature-cohort bad rate is stable."`, `root_cause_code = RC-POPULATION-MIX-SHIFT`, `severity = medium`, `created_by_agent_id = AGT-INVESTIGATION`.

Contributing factors (`fact_finding_contributing_factor`):

| `root_cause_code` | `root_cause_category` | `contribution_weight` |
|---|---|---|
| `RC-POPULATION-MIX-SHIFT` | population | primary |
| `RC-BUREAU-DATA-MISSINGNESS` | data_quality | secondary |
| `RC-POLICY-SCOPE-CHANGE` | policy | minor |

Evidence links (`fact_finding_evidence_link`): all nine evidence items linked to `FIND-2026-0031-01`; `EVID-2026-0031-08` linked with `link_role = contradictory` (against the rejected H4, retained as part of the finding's audit trail even though H4 was not the conclusion).

## 6. Critic review

Assurance/Critic Agent checks (per `agents/ASSURANCE_AGENT.md`):
- ✅ Every claim in the finding statement traces to ≥1 evidence item.
- ✅ All evidence dates fall within the June 2026 run period or its documented reference baseline.
- ✅ `root_cause_code`s exist in `dim_root_cause_taxonomy`.
- ✅ Model version referenced (`CC-BSCORE-v2.1`) matches the deployment record that actually produced the breached metric (`fact_model_deployment`, `deployment_status = active`) — no version mismatch (contrast with Scenario 004).
- ✅ At least one counter-evidence search was attempted and is documented (`EVID-2026-0031-08`).

`fact_agent_evaluation`: `evaluation_id = EVAL-2026-0031-01`, `grounding_pass = true`, `citation_pass = true`, `outcome = pass`, `evaluator = AGT-ASSURANCE`.

`fact_human_approval`: `approval_id = APR-2026-0031-01`, `approval_stage = analyst_review`, `approved_by = "N. Thanakit (Model Risk Analyst)"`, `approval_outcome = approved`.

## 7. Recommendation

`fact_recommendation`: `recommendation_id = REC-2026-0031-01`, `finding_id = FIND-2026-0031-01`, `recommendation_type = monitor`, `recommendation_text = "Continue monthly monitoring at current cadence; add a standing segment-level PSI cut for thin-file customers to distinguish future genuine drift from expected campaign-cohort maturation; open a data-quality remediation ticket for bureau feed BRU-FEED-07; note the referral-rule change in the population definition documentation so future PSI baselines account for it."`, `proposed_owner = "Retail Risk Analytics"`, `requires_committee_approval = true`, `status = approved`.

## 8. Committee decision

`fact_committee_decision`: `decision_id = DEC-2026-0014`, `investigation_id = INV-2026-0031`, `recommendation_id = REC-2026-0031-01`, `decision = approve`, `decision_rationale = "Evidence trace supports mix-shift and bureau-missingness explanation; PSI expected to normalize as campaign cohort matures and referral-rule baseline is incorporated. Approve recommendation as written; require DQ ticket closure confirmation within 30 days."`, `decided_by = "J. Sirisak (MRC Chair)"`, `committee_name = "Model Risk Committee"`, `decision_date = 2026-07-22`.

`fact_investigation.status` → `closed`, `closed_date = 2026-07-22`, `decision_id = DEC-2026-0014`.

## 9. Remediation

`fact_remediation_action`: `remediation_action_id = REM-2026-0031-01`, `recommendation_id = REC-2026-0031-01`, `decision_id = DEC-2026-0014`, `action_description = "Bureau data engineering to fix upstream feed BRU-FEED-07 missingness spike"`, `accountable_owner = "Data Engineering — Bureau Integration"`, `target_completion_date = 2026-08-21`, `status = in_progress` (closure evidence to be attached to `closure_evidence_item_id` once complete — demonstrated as `in_progress` in the MVP demo, since actual completion is outside the demo's time horizon).

## 10. Full trace query (what the Reporting Agent actually runs to build the committee report)

```
fact_investigation (INV-2026-0031)
  ⋈ fact_monitoring_metric (trigger, monitoring_metric_sk=810234)
  ⋈ fact_evidence_item (9 rows, investigation_id=INV-2026-0031)
  ⋈ fact_finding (FIND-2026-0031-01)
  ⋈ fact_finding_evidence_link (9 links)
  ⋈ fact_finding_contributing_factor (3 rows)
  ⋈ dim_root_cause_taxonomy (3 lookups)
  ⋈ fact_recommendation (REC-2026-0031-01)
  ⋈ fact_committee_decision (DEC-2026-0014)
  ⋈ fact_remediation_action (REM-2026-0031-01)
```

This exact join is what `demo/SAMPLE_EVIDENCE_LEDGER.md` renders in human-readable form, and what `demo/SAMPLE_COMMITTEE_REPORT.md` summarizes for the committee pack.
