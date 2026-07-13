# Data Test Cases

**Audience:** whoever builds the synthetic data generator and warehouse loader.

## Referential integrity
| ID | Check | Expected |
|---|---|---|
| DATA-01 | Every `dim_account.customer_sk` resolves to a `dim_customer` row | 100% resolve |
| DATA-02 | Every `fact_account_snapshot_monthly.account_sk` resolves to a current or historical `dim_account` row | 100% resolve |
| DATA-03 | Every `dim_account` with `product_code IN ('CC','SPC')` has exactly one `dim_account_revolving_ext` row | 100% match, zero orphans, zero for SPL |
| DATA-04 | Every `dim_account` with `product_code = 'SPL'` has exactly one `dim_account_termloan_ext` row | 100% match, zero for CC/SPC |

## Temporal logic
| ID | Check | Expected |
|---|---|---|
| DATA-05 | Every `fact_model_score_event` with `model_family='B'` has `mob_at_observation >= 6` | 100% (any violation = a deliberately planted Scenario 005 fixture, otherwise zero) |
| DATA-06 | Every `fact_performance_label` with `performance_window_end < current_processing_date` has `maturity_status IN ('mature','censored')`, never `immature` | 100% |
| DATA-07 | Every `fact_feature_lineage.feature_value`'s source row event time ≤ its score event's `observation_date` | 100% (leakage check) |
| DATA-08 | MOB computed from `booking_date`/`snapshot_date` matches `fact_account_snapshot_monthly.mob` exactly | 100% |

## Product-extension pattern
| ID | Check | Expected |
|---|---|---|
| DATA-09 | Zero rows in `dim_account_revolving_ext` for SPL accounts | 0 |
| DATA-10 | Zero rows in `dim_account_termloan_ext` for CC/SPC accounts | 0 |

## Censoring logic
| ID | Check | Expected |
|---|---|---|
| DATA-11 | Accounts closed (`closed_paid`) before `performance_window_end` have `maturity_status='censored'`, `censor_reason='account_closed'`, `outcome_value` null | 100% |
| DATA-12 | SPL accounts fully prepaid before `performance_window_end` have `maturity_status='censored'`, `censor_reason='paid_off'` | 100% |
| DATA-13 | Charge-off events have `maturity_status='mature'` from the charge-off date forward, regardless of window elapsed | 100% |

## Scenario 001 fixture integrity
| ID | Check | Expected |
|---|---|---|
| DATA-14 | 1,340 applications tagged `campaign_id='CMP-2025-THINFILE-04'` exist, booked Dec 2025 | Exact count |
| DATA-15 | Thin-file segment share rises from ~8% (May) to ~18% (June) in the synthetic segment metrics | Within ±1pp of scenario narrative |
| DATA-16 | Bureau missingness for June is elevated (~0.42) vs. rolling baseline (~0.11) for the affected feed | Within tolerance |
