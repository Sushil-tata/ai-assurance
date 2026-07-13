# Monitoring Data Mart (Part 5)

**Audience:** Model Risk Architect, Azure Function/pipeline builders, Monitoring Agent design owner.
**Governing principle:** 1.1 (deterministic calculation). Every metric below is computed by a pipeline and written to `fact_monitoring_metric` or `fact_segment_monitoring_metric`; no agent computes any of these values itself.
**Hard precondition for every metric in this file:** input population is filtered to `fact_performance_label.maturity_status = 'mature'` wherever the metric depends on an outcome label (population counts and drift metrics that don't need an outcome, e.g. PSI on score distribution, are the documented exception — see per-metric notes).

## 1. Metric catalog

### 1.1 Population counts
- **Formula:** `COUNT(DISTINCT account_id)` from `fact_model_score_event` for the model version and run period.
- **Required input:** `fact_model_score_event` (filtered `run_type = 'live'`).
- **Reference population:** prior period's count (for trend), or development sample count (for baseline comparison).
- **Current population:** current run period's scored accounts.
- **Segment dimensions:** product, channel, vintage, thin-file flag.
- **Minimum sample:** N/A (this metric itself informs sufficiency of others).
- **Maturity condition:** none — population count uses all scored accounts, mature or not, because it answers "how many did we score," not "how many resolved."
- **Threshold logic:** informational; RAG only applies if count drops >30% period-over-period (potential pipeline failure signal, cross-referenced with DQ pipeline-completeness check).
- **Owner:** Monitoring Agent / Retail Risk Analytics.
- **Applies to:** A, B, C Score equally.

### 1.2 Score distribution
- **Formula:** histogram of `score_value` in governed bins (from `dim_governance_threshold` bucket definitions).
- **Required input:** `fact_model_score_event.score_value`.
- **Maturity condition:** none (distributional, not outcome-based).
- **Applies to:** A, B, C — same formula, different bin definitions per model.

### 1.3 Approval / decline / treatment rate
- **Formula:** `COUNT(decision = 'approved') / COUNT(*)` (A Score, from `fact_application.decision`); for B/C Score, the analogous concept is **treatment rate** — `COUNT(collections_treatment_code IS NOT NULL) / COUNT(*)` from `fact_delinquency_state_monthly`.
- **Differs by family:** A Score only has approval/decline; B Score has no equivalent (it doesn't gate a decision, it scores an existing account); C Score has treatment rate instead.
- **Maturity condition:** none.
- **Applies to:** A Score (approval/decline), C Score (treatment rate). **Not applicable to B Score.**

### 1.4 Bad rate
- **Formula:** `COUNT(outcome_value IN bad_definition_set) / COUNT(*)` from `fact_performance_label` **WHERE `maturity_status = 'mature'`**.
- **Required input:** `fact_performance_label`, `dim_governance_threshold` (bad definition).
- **Reference population:** development sample bad rate.
- **Current population:** current mature cohort.
- **Minimum sample:** 500 mature observations (below this, `statistical_reliability_flag = false`).
- **Maturity condition:** **mandatory** — this is the metric ADR-006 exists to protect.
- **Applies to:** A, B, C — same formula, different `bad_definition_id` per model family (A: 12-month contractual default; B: 6-month roll to 60+ DPD; C: bucket-specific roll-forward/charge-off).

### 1.5 Gini / AUC
- **Formula:** standard Gini coefficient from the ROC curve of `score_value` vs. `outcome_value` (binarized per `bad_definition_id`), computed via the Azure Function statistical wrapper (`architecture/SYSTEM_ARCHITECTURE.md` §1).
- **Required input:** `fact_model_score_event.score_value` joined to `fact_performance_label.outcome_value`, **mature only**.
- **Minimum sample:** 1,000 mature observations with both classes represented (≥ 30 bads).
- **Maturity condition:** mandatory.
- **RAG logic:** Green if Gini within 10% relative of validation-sample Gini; Amber 10–25% relative degradation; Red >25% or below `dim_governance_threshold.green_upper_bound`.
- **Applies to:** A, B, C.

### 1.6 KS (Kolmogorov-Smirnov)
- **Formula:** max separation between cumulative good/bad distributions across score bins.
- Same input/maturity/sample requirements as Gini.
- **Applies to:** A, B, C.

### 1.7 Lift
- **Formula:** bad rate in top decile / overall bad rate, computed on mature population.
- **Applies to:** A, B, C. For C Score, "top decile" is interpreted as highest-risk decile (roll-forward propensity), same mechanics.

### 1.8 Calibration bins / observed-to-expected
- **Formula:** for each score band, `observed_bad_rate / expected_bad_rate` (expected = validation-sample bad rate for that band).
- **Required input:** `fact_performance_label` (mature), `fact_model_score_event.score_band`.
- **Minimum sample:** 100 mature observations per band (bands below this are shown but flagged unreliable, not suppressed).
- **Maturity condition:** mandatory.
- **RAG logic:** Green O/E within [0.85, 1.15]; Amber [0.7, 0.85) or (1.15, 1.3]; Red outside that.
- **Applies to:** A, B, C.

### 1.9 Brier score
- **Formula:** mean squared error between `score_value` (as probability) and binary `outcome_value`, mature population only.
- **Applies to:** A, B, C — most meaningful for B/C given their shorter, more frequent horizons.

### 1.10 PSI (Population Stability Index)
- **Formula:** `Σ (current_pct - reference_pct) * ln(current_pct / reference_pct)` across governed score bins.
- **Required input:** `fact_model_score_event.score_value` distribution, current run vs. reference (development sample or prior period, per `dim_governance_threshold` configuration).
- **Maturity condition:** **none** — PSI is a distributional stability check on the *scored population*, not an outcome metric; it is deliberately computable on immature scores, which is what makes it an *early warning* metric (this is why Scenario 001 catches the issue via PSI weeks before bad-rate metrics would even have a mature population to compute on).
- **Minimum sample:** 1,000.
- **RAG logic:** from `dim_governance_threshold` — Green < 0.10, Amber 0.10–0.25, Red > 0.25 (industry-standard bands, configured not hardcoded).
- **Applies to:** A, B, C — computed on the score distribution in every case. **This is the metric in Scenario 001.**

### 1.11 CSI (Characteristic Stability Index)
- **Formula:** same PSI formula, applied per input feature (`dim_feature_definition`) rather than the final score.
- **Required input:** `fact_feature_lineage.feature_value` distributions, current vs. reference.
- **Maturity condition:** none.
- **Applies to:** A, B, C — this is what lets the Investigation Agent decompose a score-level PSI breach down to "which feature moved" (Scenario 001 cause 1 and 2 are both found via CSI decomposition).

### 1.12 Feature drift / missingness
- **Formula:** missingness = `COUNT(is_missing = true) / COUNT(*)` per feature per run, from `fact_feature_lineage`.
- **Maturity condition:** none.
- **RAG logic:** Green < 5% missing, Amber 5–15%, Red > 15% (bureau features specifically — see `bureau_missingness_pct` in `fact_bureau_aggregate_monthly`, which feeds this metric for bureau-sourced features).
- **Applies to:** A, B, C — any feature set.

### 1.13 Category drift
- **Formula:** PSI-style formula applied to categorical feature level distributions (e.g., `employment_status`, `application_channel`).
- **Applies to:** A Score primarily (categorical acquisition features); B/C to a lesser extent (mostly continuous behavioural features).

### 1.14 Segment performance
- Every metric in 1.4–1.11 additionally computed at `dim_monitoring_segment` grain, written to `fact_segment_monitoring_metric` instead of `fact_monitoring_metric`. Same formulas, `current_population_def` scoped to the segment.

### 1.15 Vintage performance
- Bad rate, Gini, KS computed per booking-month vintage cohort — requires vintage alignment (`TEMPORAL_MODEL.md` §5.2) so cohorts are compared at equal MOB, not equal calendar date.

### 1.16 MOB performance
- Bad rate and score distribution computed per MOB value (MOB 6, 7, 8...) — this is the primary lens for B Score specifically, since B Score's whole premise is MOB-conditioned eligibility.

### 1.17 Roll rates (revolving and delinquency generally, C Score primary)
- **Formula:** `COUNT(bucket_current = n+1 WHERE bucket_prior_month = n) / COUNT(bucket_prior_month = n)` from `fact_delinquency_state_monthly`.
- **Maturity condition:** transition itself is always "mature" the moment it's observed (it's not a forward-looking label) — no filter needed beyond the snapshot existing.
- **Applies to:** C Score primarily; also used as revolving-product-specific collections signal.

### 1.18 Cure rates
- **Formula:** `COUNT(transition_type = 'cure') / COUNT(bucket_prior_month IS NOT NULL AND bucket_prior_month > 0)`.
- **Applies to:** C Score.

### 1.19 Redefault
- **Formula:** among cured accounts, `COUNT(re-entered delinquency within N months of cure) / COUNT(cured)`.
- **Requires:** a forward observation window on the cure event itself (same maturity discipline as §1.4, applied to the cure sub-population).
- **Applies to:** C Score.

### 1.20 Override rate
- **Formula:** `COUNT(fact_agent_override) / COUNT(fact_committee_decision)` over a period.
- **Required input:** `fact_agent_override`, `fact_committee_decision`.
- **Maturity condition:** N/A (governance metric, not a model-performance metric).
- **Applies to:** all — this is an **agent-quality** metric, not a model metric; it belongs in the mart because it is computed the same deterministic way, but its subject is AI Assurance itself, not the underlying credit model. See `agents/AGENT_EVALUATION.md`.

### 1.21 Model applicability violations
- **Formula:** `COUNT(fact_model_score_event WHERE model_family = 'B' AND mob_at_observation < 6)` — should always be **zero**; any non-zero count is itself a Red-severity Data Quality Agent finding, not a monitoring metric with a RAG band.
- **Applies to:** B Score specifically (the MOB ≥ 6 rule); analogous applicability checks exist for A Score (`observation_date` must precede `booking_date`) and C Score (`delinquency_bucket_id` must be non-null and match the model subtype).

### 1.22 Model version mismatches
- **Formula:** `COUNT(fact_model_score_event WHERE model_version_id NOT IN (SELECT model_version_id FROM fact_model_deployment WHERE deployment_status = 'active' AND environment = 'production'))` — see Scenario 004.
- **Applies to:** all families.

### 1.23 Data-source latency
- **Formula:** `AVG(load_timestamp - event_time)` per source table per run, compared against each source's documented SLA.
- **Applies to:** all — feeds the Data Quality Agent, not model-specific.

### 1.24 Pipeline completeness
- **Formula:** `COUNT(accounts expected to have a snapshot this month) - COUNT(accounts with an actual snapshot row)`; expected count from `dim_account` active population.
- **Applies to:** all.

### 1.25 Sample-size sufficiency
- **Formula:** flag, not a metric in its own right — `statistical_reliability_flag = (sample_size_current >= minimum_sample_required)`, attached to every metric row above per §1.4–1.11's per-metric minimums.
- **Applies to:** all.

## 2. What differs by model family — summary table

| Metric | A Score | B Score | C Score |
|---|---|---|---|
| Approval/decline rate | Yes | N/A | N/A (treatment rate instead) |
| MOB performance | N/A (pre-booking) | **Primary lens** | Secondary |
| Bucket-specific bad definition | N/A | N/A | **Required, bucket-specific** |
| Roll rate / cure rate / redefault | N/A | N/A | **Primary metrics** |
| Vintage performance | Primary lens | Secondary | Secondary |
| PSI/CSI | Yes | Yes | Yes |

## 3. What differs by product family

| Metric | Revolving (CC, SPC) | Term loan (SPL) |
|---|---|---|
| Utilization/payment-ratio drift | Yes (CSI on `fact_revolving_behaviour_monthly` fields) | N/A |
| Maturity-stage segment performance | N/A | **Yes** — `fact_termloan_behaviour_monthly.maturity_stage` is a standing segment dimension |
| Prepayment-adjusted bad rate | N/A | **Yes** — censoring treatment per `TEMPORAL_MODEL.md` §5.5 changes the effective denominator |
| Near-maturity horizon flag | N/A | **Yes** — per `TEMPORAL_MODEL.md` §7.5 |

## 4. RAG status computation (applies to every metric with a threshold)

```
rag_status =
  CASE
    WHEN NOT statistical_reliability_flag THEN 'insufficient_sample' -- not a RAG color; surfaced distinctly
    WHEN metric_value <= threshold.green_upper_bound THEN 'green'
    WHEN metric_value <= threshold.amber_upper_bound THEN 'amber'
    ELSE 'red'
  END
```
Threshold values are looked up from `dim_governance_threshold` at the `run_date`, using the threshold row effective on that date — never a hardcoded constant (ADR-007).
