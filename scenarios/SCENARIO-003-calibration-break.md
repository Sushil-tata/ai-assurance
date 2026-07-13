# Scenario 003 — Calibration Break (Observed-to-Expected Drift)

**Status:** enterprise/regression test scenario.

## Situation
`CC-BSCORE-v2.1`'s calibration O/E for score band B3 drifts to 1.42 (Red) in August 2026 while PSI/CSI remain Green — the score distribution hasn't shifted, but realized bad rates within a specific band have risen.

## Trigger
`fact_monitoring_metric`: `metric_code = CALIBRATION_OE`, band B3, `metric_value = 1.42`, `rag_status = red`.

## Hypotheses tested
1. Genuine economic/behavioural deterioration concentrated in band B3 (supported: segment cut shows the effect is broad-based across product/channel, not isolated to one campaign or DQ issue).
2. A bad-definition change altering what counts as "bad" (rejected: `bad_definition_id` unchanged for the period, confirmed via Governance Agent).
3. A scoring feature computation bug specific to band B3's typical feature ranges (tested via CSI decomposition — no individual feature shows anomalous drift).

## Finding
Genuine calibration drift in band B3, likely reflecting real credit-quality change in that risk band, not a data or population artifact. Root cause: `RC-GENUINE-CALIBRATION-DRIFT` (category: `model`).

## Recommendation
`recommendation_type = recalibration` — escalates to Model Development Team, requires full Model Risk Committee sign-off per `policies/ESCALATION_MATRIX.md` (recalibration always requires committee, no analyst-only path).

## What this scenario tests
That the system correctly distinguishes a genuine model-quality signal (which *should* trigger the heavier recalibration recommendation path) from the population/data-quality artifacts in Scenario 001 — i.e., that the Investigation Agent doesn't default to "it's probably just population mix" as a lazy explanation for every breach.
