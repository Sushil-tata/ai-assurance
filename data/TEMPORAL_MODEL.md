# Temporal Model (Part 3)

**Audience:** data architects, pipeline builders, model risk validators.
**Governing ADR:** `adr/ADR-006-temporal-label-maturity.md`.
**This is the second-most safety-critical document in the repository, after `TABLE_CATALOG.md`'s `fact_performance_label` spec.**

## 1. The full date-field inventory

| Date field | Table | Time type | Meaning |
|---|---|---|---|
| `application_date` | `fact_application` | Event | When the customer applied — **A Score observation point** |
| `booking_date` | `dim_account` | Event | When the account was opened/disbursed — **MOB = 0 basis** |
| `snapshot_date` | `fact_account_snapshot_monthly` | Event | Month-end account state date — **B Score observation point candidate** |
| `score_date` | `fact_model_score_event` | Event (with processing lag) | When a score was actually computed; may lag `observation_date` by the scoring pipeline's SLA |
| `validation_date` | `dim_model_version` | Event | When the model version was validated — **model effective date basis** |
| `feature_observation_date` (conceptual — implemented as `observation_lag_days` offset from `observation_date` in `dim_feature_definition` / realized per-event in `fact_feature_lineage`) | derived | Event | The as-of date a feature value reflects — must never be after `observation_date` |
| `bureau_pull_date` | `fact_bureau_tradeline` | Event | When the bureau file was pulled |
| `run_date` | `fact_monitoring_metric` | Processing | **Monitoring run date** — when the deterministic calculation executed |
| `performance_window_start` / `performance_window_end` | `fact_performance_label` | Event | Start = `observation_date`; end = start + window |
| `maturity_status` transition (implicit — the date at which a label *becomes* mature is `performance_window_end`, computed at read time against current processing date) | `fact_performance_label` | Derived | **Label maturity date** |
| `effective_date` | `dim_policy_document` | Event | **Policy effective date** |
| `effective_start_date` / `effective_end_date` | `dim_governance_threshold` | Event | **Threshold effective date** |
| `opened_date` / `closed_date` | `fact_investigation` | Event | **Investigation opened/closed dates** |
| `decision_date` | `fact_committee_decision` | Event | **Committee decision date** |
| `load_timestamp` | every table | Processing | When AI Assurance ingested/computed the row — never used in business logic |

## 2. Event time vs. processing time vs. effective time vs. system time

| Concept | Definition | Example in this schema |
|---|---|---|
| **Event time** | When the real-world thing happened | A customer made a payment on `2026-06-28` (`transaction_date`) |
| **Processing time** | When AI Assurance's pipeline ingested or computed the record | That payment was posted/loaded into the warehouse on `2026-06-29` (`posting_date` / `load_timestamp`) — a one-day lag is normal batch latency |
| **Effective time** | The date range during which a dimension row is the "current truth" (SCD2) | A customer's `income_band` row is effective `2024-03-01` to `null` (current) |
| **System time** | Pure database insert/update timestamp, technical only | `load_timestamp` on any row — used for pipeline debugging, never for computing MOB or a metric |

**Rule:** every monitoring metric, every label, every feature is computed **as of event time**, never as of processing time. This is Principle 1.2 and is the direct fix for the failure mode in `adr/ADR-006`.

## 3. Late-arriving data and restatement

**Scenario:** the June bureau file (`fact_bureau_tradeline`, `bureau_pull_date = 2026-06-05`) is reprocessed on `2026-06-20` because the original file had a partial outage — 15 days late arriving relative to its normal 2-day pipeline SLA.

**Handling:**
1. The corrected rows load with `bureau_pull_date` unchanged (`2026-06-05`, the true event time) and a new `load_timestamp` (`2026-06-20`).
2. `fact_bureau_aggregate_monthly` for `aggregation_month = 2026-06-01` is **recomputed**, not appended blindly — a new row is written with `calculation_version` incremented, and the prior version's row is retained (not deleted) for auditability.
3. Any `fact_monitoring_metric` row already published against the old aggregate is **not silently overwritten** — a new `calculation_run_id` is created for the restated run, and the investigation/report layer explicitly notes "this run restates RUN-MON-2026-06 due to late bureau data" as an evidence item if an investigation references it.

This is why `fact_monitoring_metric` and `fact_bureau_aggregate_monthly` are append-only with version fields rather than update-in-place tables — restatement must be visible, not invisible.

## 4. Back-scoring

**Definition:** running a *current* model version against *historical* observation dates, to build a like-for-like comparison baseline (e.g., "what would v2.1 have scored the January 2026 cohort?").

**Handling:** every such run writes to `fact_model_score_event` with `run_type = 'backscore'`, `score_date` = the date the backscore batch actually ran, `observation_date` = the historical date being backscored. Monitoring pipelines exclude `run_type = 'backscore'` rows from live population/RAG calculations by default — they exist for validation/challenger comparison only, and are pulled in explicitly (e.g., by the Governance Agent comparing champion vs. challenger) rather than accidentally polluting live monitoring.

## 5. Vintage alignment, MOB, maturity, censoring — the core mechanics

### 5.1 MOB calculation
`mob = DATEDIFF(month, dim_account.booking_date, fact_account_snapshot_monthly.snapshot_date)`, integer, booking month = MOB 0.

### 5.2 Vintage
An account's vintage = `DATE_TRUNC(month, booking_date)`. Used to align accounts booked in different months when comparing "MOB 6 performance" fairly — an account booked in January 2026 reaches MOB 6 in July 2026; an account booked in March 2026 reaches MOB 6 in September 2026. Monitoring segment cuts by vintage let the Investigation Agent distinguish "this cohort is genuinely worse" from "this cohort just hasn't matured yet."

### 5.3 Performance maturity
`maturity_status = 'mature'` when `current_processing_date >= performance_window_end` **and** the account was continuously observable through that date. Otherwise `'immature'` (window hasn't elapsed) or `'censored'` (window elapsed but the account stopped being observable early — closed, paid off, or charged off before `performance_window_end` in a way that makes the intended outcome unknowable).

### 5.4 Closed-account treatment
An account that closes in good standing (`account_status = 'closed_paid'`) before its performance window ends is **censored, not "good."** Treating early closure as an automatic "good" outcome is a classic survivorship bias — closure is silent about what would have happened had the account stayed open. `censor_reason = 'account_closed'` distinguishes this from maturity.

### 5.5 Prepayment treatment (SPL-specific)
A term loan that is fully prepaid before `performance_window_end` is censored with `censor_reason = 'paid_off'`, exactly like closed-account treatment above — prepayment is not evidence of "good" behaviour for scoring purposes, and folding it into "good" would bias the bad rate downward (prepayers are removed from the risk pool precisely because they had the means to pay off, which is correlated with, not identical to, low risk).

### 5.6 TDR / restructure treatment
An account that is restructured (`fact_delinquency_state_monthly.transition_type = 'restructure'` or `fact_termloan_behaviour_monthly.is_restructured = true`) during the performance window is **not censored** — restructuring is itself a defined bad-adjacent outcome captured in `outcome_value`, because it represents the credit relationship's actual trajectory, not a loss of observability. This is the key distinction from closure/prepayment: restructuring is an *outcome*, closure/prepayment is a *loss of ability to observe the outcome*.

### 5.7 Charge-off treatment
Charge-off (`fact_delinquency_state_monthly.is_charged_off = true`, `charge_off_date` populated) is the clearest "bad" outcome and is never censored — it is a definite, mature-as-of-occurrence outcome regardless of whether the nominal performance window has fully elapsed (an account charged off at month 4 of a 6-month window has already reached its worst-case outcome; the label is set to `'charge_off'` and `maturity_status = 'mature'` from the charge-off date forward, not held `'immature'` until month 6).

## 6. Feature leakage control (the observation-window / performance-window boundary)

**Hard rule, enforced by Data Quality Agent rule DQ-002:** for every row in `fact_feature_lineage`, the source data used to compute `feature_value` must have an event-time timestamp `<= score_event.observation_date`. If a feature's source row's event time falls inside or after the performance window, the pipeline run fails validation and the score event is quarantined (not silently scored) — this is the direct implementation of design principle 1.3.

## 7. Worked examples

### 7.1 A Score with 3/6/12-month labels

Customer applies `2025-11-03` (`application_date`), booked `2025-11-10` (`booking_date`). A Score `observation_date = 2025-11-03` (application date, pre-booking).

| Window | `performance_window_start` | `performance_window_end` | Status as of `2026-07-13` (today) |
|---|---|---|---|
| 3-month | 2025-11-03 | 2026-02-03 | Mature (elapsed) |
| 6-month | 2025-11-03 | 2026-05-03 | Mature (elapsed) |
| 12-month | 2025-11-03 | 2026-11-03 | **Immature** — window has not elapsed; excluded from any monitoring metric run today |

### 7.2 B Score at MOB 6, 7 and 8 (same account, three monthly score events)

Account booked `2025-11-10`. B Score is ineligible before MOB 6.

| Snapshot | `snapshot_date` | `mob` | B Score eligible? | `score_event_id` created? |
|---|---|---|---|---|
| MOB 5 | 2026-04-30 | 5 | No (`mob < 6`) | No — no score event row at all, not a null/excluded row |
| MOB 6 | 2026-05-31 | 6 | Yes | `SCORE-CCB-20260531-ACC0091823` |
| MOB 7 | 2026-06-30 | 7 | Yes | `SCORE-CCB-20260630-ACC0091823` (the running example used throughout `TABLE_CATALOG.md`) |
| MOB 8 | 2026-07-31 | 8 | Yes | `SCORE-CCB-20260731-ACC0091823` |

Each monthly score event gets its own independent 3/6/12-month forward labels — MOB 7's 6-month window (`2026-06-30` to `2026-12-30`) is immature today (`2026-07-13`); MOB 6's 3-month window (`2026-05-31` to `2026-08-31`) is also still immature today.

### 7.3 Bucket 0 C Score

Account enters Bucket 0 (1–29 DPD) at `snapshot_date = 2026-05-31`. C Score `observation_date = 2026-05-31`, `delinquency_bucket_id = 0`. Outcome window (e.g., 3 months) asks: does this account cure, remain in Bucket 0/roll to Bucket 1+, or reach NPL/charge-off by `2026-08-31`? At `2026-06-30`, `fact_delinquency_state_monthly.transition_type = 'cure'` would set `outcome_value = 'cure'` immediately upon the cure event (mature from the cure date, doesn't need to wait for the full window if the outcome is already unambiguous under the governed bad definition — configured per `bad_definition_id`).

### 7.4 Bucket 1 C Score

Same account rolls forward instead: `bucket_prior_month = 0`, `bucket_current = 1` at `snapshot_date = 2026-06-30`. This creates a **new, independent** C Score observation (`delinquency_bucket_id = 1`, `observation_date = 2026-06-30`) — it does not overwrite or update the Bucket 0 observation from the prior month, which retains its own eventual outcome (`roll_forward`, in this case, which is itself a valid mature bad-adjacent outcome for the Bucket 0 model).

### 7.5 SPL account near maturity

SPL account, `original_tenor_months = 36`, `first_instalment_date = 2024-02-05`, `scheduled_maturity_date = 2027-01-05`. As of `snapshot_date = 2026-07-31`: `remaining_tenor_months = 6`, `maturity_stage = 'late'`. A B Score observation at this snapshot has only a 6-month runway before scheduled maturity — its **12-month performance window would extend past `scheduled_maturity_date`**. This is not an error: the account may prepay, may pay to term (closing at maturity, itself a censoring event per §5.5 if the loan simply matures on schedule with no further scheduled instalments — the pipeline treats scheduled maturity identically to prepayment for censoring purposes), or may still be delinquent and roll into collections past its nominal term. The monitoring pipeline flags any C/B Score observation whose full performance window extends past `scheduled_maturity_date` with a `near_maturity_horizon_flag` (documented in `MONITORING_MART.md`), so the Investigation Agent knows a lower mature-sample count for that vintage is expected, not anomalous.

## 8. Summary rule set (quotable, for the Data Quality Agent's rule engine)

1. No feature may reference data with event time after `observation_date`.
2. No metric may include a `fact_performance_label` row where `maturity_status != 'mature'`.
3. `run_type = 'backscore'` rows are excluded from live monitoring aggregation by default.
4. Closure, prepayment, and scheduled maturity are censoring events, not "good" outcomes.
5. Restructuring and charge-off are outcomes, not censoring events, and are mature from their occurrence date.
6. Every restatement creates a new versioned row; nothing is silently overwritten.
