# Remediation Playbook

**Audience:** accountable owners named in `fact_recommendation.proposed_owner`, Reporting & Approval Agent design owner.

## 1. What "remediation" means in this system

A remediation action (`fact_remediation_action`) is created only after a committee decision approves a recommendation. AI Assurance **never executes** a remediation itself (Principle 1.10) — this playbook describes the human/team-owned process that happens after AI Assurance hands off.

## 2. Standard remediation types and typical owners

| `recommendation_type` | Typical accountable owner | Typical action |
|---|---|---|
| `monitor` | Retail Risk Analytics | No structural change; enhanced monitoring cadence/segment cuts added |
| `investigate_further` | Model Risk Analyst | Deeper manual analysis beyond automated investigation scope |
| `feature_fix` | Data Engineering | Upstream pipeline/feed correction (e.g., Scenario 001's bureau feed fix) |
| `recalibration` | Model Development Team | Model recalibration cycle, re-enters `dim_model.lifecycle_stage = recalibration` |
| `policy_review` | Governance / Collections Strategy | Policy clause or threshold review and potential amendment |
| `no_action` | N/A | Recorded for audit completeness even when no action is needed |

## 3. Closure requirements

Every `fact_remediation_action` must reach `status = complete` with `actual_completion_date` populated and `closure_evidence_item_id` pointing to a `fact_evidence_item` row demonstrating completion (e.g., a DQ check subsequently passing, a new monitoring run showing normalized PSI). Closure without evidence is not permitted — the Reporting & Approval Agent's weekly sweep flags any remediation marked complete without a linked evidence item as an audit exception.

## 4. Example (Scenario 001)

`REM-2026-0031-01`: "Bureau data engineering to fix upstream feed BRU-FEED-07 missingness spike," owner Data Engineering — Bureau Integration, target `2026-08-21`. Expected closure evidence: a subsequent `fact_data_quality_check` row for `rule_code = DQ-014` showing `result_status = pass` with missingness back under the 5% Green band, linked as `closure_evidence_item_id`.

## 5. Overdue handling

See `policies/ESCALATION_MATRIX.md` — overdue remediation escalates to the accountable owner and MRC chair within 1 business day of the missed `target_completion_date`.
