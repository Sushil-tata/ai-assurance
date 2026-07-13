# Scenario 002 — Pipeline Non-Completion (Data Quality Failure)

**Status:** enterprise/regression test scenario, not the MVP primary demo (that's Scenario 001), but designed for build-time testing of the Monitoring Agent's failure-handling path.

## Situation
The July 2026 monitoring pipeline run fails to complete for `CC-BSCORE-v2.1` — `fact_account_snapshot_monthly` is only 60% populated for the expected active-account population when the scheduled monitoring run fires, due to an upstream servicing-extract delay.

## Trigger
Monitoring Agent's scheduled run finds no `fact_monitoring_metric` rows for `calculation_run_id = RUN-MON-2026-07` at the expected run time. Per `agents/MONITORING_AGENT.md` §"Error handling," this does not get reported as "all Green" — it opens `fact_investigation` with `trigger_type = dq_failure`, `severity = high`.

## Evidence
- `fact_data_quality_check`: `rule_code = DQ-001` (pipeline completeness), `result_status = fail`, `result_value = 0.60`, `affected_row_count` = expected active population minus actual snapshot rows.
- `fact_data_quality_check`: `rule_code = DQ-023` (source latency), showing the servicing extract landed 3 days late against SLA.

## Finding
"July monitoring metrics could not be computed on schedule because the underlying account snapshot pipeline was only 60% complete at run time, due to a 3-day-late servicing extract. This is a data pipeline failure, not a model or population signal — no PSI/bad-rate conclusion can be drawn from partial data." Root cause: `RC-PIPELINE-INCOMPLETE` (category: `data_quality`).

## Recommendation
`recommendation_type = feature_fix` (in the sense of pipeline fix, not a scoring feature) — reschedule the monitoring run once the extract completes; escalate the extract SLA breach to Data Engineering; no monitoring conclusion is drawn for July until data is complete.

## Committee decision
Approve; require re-run confirmation before the July cycle is considered closed.

## What this scenario tests
That the Monitoring Agent never silently reports "no breach" when it actually has no valid data to evaluate — the single most dangerous failure mode for a monitoring system (a false negative caused by infrastructure, not by the underlying model actually being fine).
