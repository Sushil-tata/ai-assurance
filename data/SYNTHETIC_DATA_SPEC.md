# Synthetic Data Specification

**Audience:** whoever builds the data generator (see `DELIVERY_PLAN.md` step 4).
**Principle:** synthetic but realistic — every distribution and edge case a real monitoring system would encounter must be represented, including the three engineered causes in Scenario 001.

## 1. Volume (demo/UAT environment)

| Entity | Count | Notes |
|---|---|---|
| Customers | 20,000 | |
| Credit Card accounts | 18,000 | MVP live product |
| Speedy Cash accounts | 4,000 | Registry-populated, no live monitoring in MVP |
| Speedy Loan accounts | 3,000 | Registry-populated, no live monitoring in MVP |
| Applications | 25,000 | Includes declines |
| Monthly snapshots | ~18 months history × active accounts | Booking dates spread Jan 2024 – Jun 2026 |
| Bureau tradelines | ~3 per customer per pull, monthly pulls | |
| Score events | Monthly per eligible account per applicable model | CC B Score only carries live monitoring data |

## 2. Generation approach

1. **Customer generator** produces `dim_customer` with realistic income/employment/geography distributions (synthetic, no real individuals).
2. **Application generator** produces `fact_application` with a configurable channel mix, including a deliberate campaign cohort (`CMP-2025-THINFILE-04`, 1,340 applications, thin-file-skewed, booked December 2025) for Scenario 001.
3. **Account generator** creates `dim_account` + product extension rows from approved applications, assigning `product_family` correctly (zero cross-contamination between revolving/term-loan extension tables — see `testing/DATA_TEST_CASES.md` DATA-09/10).
4. **Monthly snapshot generator** walks each account forward month by month from `booking_date`, generating `fact_account_snapshot_monthly` + the correct product extension (`fact_revolving_behaviour_monthly` or `fact_termloan_behaviour_monthly`) + `fact_delinquency_state_monthly` where applicable, with realistic autocorrelated behaviour (an account with rising utilization one month is more likely to have rising utilization the next).
5. **Bureau generator** produces `fact_bureau_tradeline` monthly pulls, with a deliberately injected missingness spike (`BRU-FEED-07`, 2026-06-03 to 2026-06-09) for Scenario 001, otherwise a stable ~11% baseline missingness rate.
6. **Scoring pipeline (synthetic)** produces `fact_model_score_event` for CC B Score only (live), applying the MOB ≥ 6 eligibility filter correctly, plus registry-only score events are NOT generated for the other 11 models (no live data, per `SCOPE.md`).
7. **Label constructor** produces `fact_performance_label` with correct maturity/censoring logic per `data/TEMPORAL_MODEL.md` §5, run against "today" = 2026-07-13 so a realistic mix of mature/immature/censored labels exists.
8. **Monitoring pipeline (synthetic)** computes `fact_monitoring_metric` / `fact_segment_monitoring_metric` for CC B Score, deliberately landing PSI at 0.263 for June 2026 by construction (the campaign cohort + bureau missingness + referral rule change are the actual generative causes, not a hardcoded metric value — the PSI figure should emerge from the underlying population changes when the real PSI formula is applied, not be hand-set).

## 3. Engineered realism requirements

- Bad rates must be internally consistent with score bands (higher score band → lower observed bad rate, on average, with realistic noise) — a monitoring demo with an incoherent score-to-outcome relationship undermines its own credibility.
- MOB-eligible population must show believable growth as campaign cohorts mature into the scoring population over time — this is what makes Scenario 001's segment-share evidence (8.1% → 18.4%) a plausible number, not an arbitrary one.
- Term-loan (SPL) accounts must show realistic amortization schedules and a distribution of maturity stages (early/mid/late/matured) for future enterprise-scope demonstration.

## 4. What is explicitly NOT randomized

The three Scenario 001 causes (campaign cohort size and timing, bureau feed outage window, referral-rule change date) are fixed parameters, not randomly seeded — the demo must be reproducible exactly, every run, not "usually shows something similar."

## 5. Refresh cadence
Regenerated on demand during build (`DELIVERY_PLAN.md` step 4); a full regeneration must complete in under 30 minutes on a standard dev environment to keep the build loop fast.
