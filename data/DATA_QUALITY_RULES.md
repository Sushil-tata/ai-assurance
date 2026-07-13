# Data Quality Rules

**Audience:** pipeline builders, Data Quality Agent design owner.
**Source of truth for results:** `fact_data_quality_check` (Domain 18). This file defines the rules that populate it.

| Rule code | Target table | Check type | Logic | Fail action |
|---|---|---|---|---|
| DQ-001 | `fact_account_snapshot_monthly` | completeness | `COUNT(actual snapshots) / COUNT(expected active accounts) >= 0.98` | Blocks monitoring run for the period; opens `dq_failure` investigation (Scenario 002) |
| DQ-002 | `fact_feature_lineage` | referential / leakage | Source row event time ≤ `score_event.observation_date` for every feature | Quarantines the affected score event, does not score it |
| DQ-003 | `dim_account_revolving_ext` | referential | Zero rows where parent `dim_account.product_code = 'SPL'` | Blocks account-generation pipeline promotion to silver layer |
| DQ-004 | `dim_account_termloan_ext` | referential | Zero rows where parent `dim_account.product_code IN ('CC','SPC')` | Same as DQ-003 |
| DQ-014 | `fact_bureau_aggregate_monthly` | missingness | `bureau_missingness_pct` vs. 90-day rolling baseline, fail if > 2x baseline | Evidence item on request; does not block the monitoring run (missingness is itself a valid, monitorable signal, not a hard stop) |
| DQ-021 | `fact_model_score_event` | applicability | Zero rows where `model_family='B' AND mob_at_observation < 6` | `severity=critical` investigation, immediate (Scenario 005) |
| DQ-022 | `fact_model_score_event` | version consistency | Zero rows where `model_version_id` not in currently active `fact_model_deployment` | `severity=critical` investigation, immediate (Scenario 004) |
| DQ-023 | all source-fed tables | latency | `load_timestamp - event_time` within documented SLA per source | Warn if within 2x SLA, fail if beyond |
| DQ-024 | `fact_performance_label` | maturity logic | Zero rows where `performance_window_end < current_date` and `maturity_status = 'immature'` (should be `mature` or `censored` by then) | Blocks label publication for affected rows, alerts pipeline owner |

## Rule severity bands
- **Blocking** (DQ-001, DQ-002, DQ-003, DQ-004, DQ-024): pipeline does not proceed to publish affected output.
- **Critical-investigation** (DQ-021, DQ-022): bypasses the normal monitoring RAG cycle, immediate investigation regardless of schedule.
- **Evidence-only** (DQ-014, DQ-023 warn tier): recorded, available to the Investigation Agent on request, does not itself block anything.

## Ownership
Retail Risk Analytics owns rule definitions; Data Engineering owns the pipeline implementations that produce `fact_data_quality_check` rows; the Data Quality Agent (per its contract) only ever *reads* these results, never computes them itself.
