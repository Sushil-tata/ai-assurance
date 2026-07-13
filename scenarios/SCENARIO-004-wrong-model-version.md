# Scenario 004 — Wrong Model Version Scoring Live Traffic

**Status:** enterprise/regression test scenario; exercises Governance/Investigation version-consistency checks referenced throughout `agents/ASSURANCE_AGENT.md`.

## Situation
A deployment rollback error leaves `CC-BSCORE-v2.0` (the predecessor version) actively scoring 15% of traffic for four days in September 2026, after `v2.1` was believed to be fully cut over.

## Trigger
Monitoring metric `metric_code = MODEL_VERSION_MISMATCH_COUNT` (per `data/MONITORING_MART.md` §1.22) returns a non-zero count: `fact_model_score_event` rows exist referencing `CC-BSCORE-v2.0` after its `fact_model_deployment.deployment_status` was set to `superseded`.

## Evidence
- `fact_model_deployment`: `deployment_id = DEPLOY-CCB-0090` (v2.0), `deployment_status = superseded`, `deployed_at` shows the supersession date.
- `fact_model_score_event`: 1,842 rows with `model_version_id = CC-BSCORE-v2.0` and `score_date` after the supersession date.

## Finding
A deployment configuration error caused 15% of scoring traffic to be served by the retired v2.0 model for 4 days. This is a **production control failure**, not a monitoring-metric problem — flagged `severity = critical` and root cause `RC-MODEL-VERSION-MISMATCH` (category: `model`), bypassing the standard Amber/Red RAG cycle entirely per the Escalation Matrix's dedicated version-mismatch row.

## Recommendation
Immediate re-verification of production routing configuration (outside AI Assurance's scope — flagged to Engineering); affected score events flagged for potential re-scoring; committee notified same day.

## What this scenario tests
That the Assurance/Critic Agent's version-consistency check (checklist item (d) in `agents/ASSURANCE_AGENT.md`) actually catches a case where the *metric itself* would otherwise look like a normal drift signal if the version mismatch weren't separately checked — a naive system might mis-attribute this to population drift exactly like Scenario 001, when the real cause is a deployment control failure.
