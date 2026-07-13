# ADR-006: Strict Label Maturity Enforcement

**Status:** Accepted
**Date:** 2026-07-13
**Deciders:** Model Risk Architect, Principal Enterprise Data Architect

## Context
The brief mandates: "No monitoring metric may use immature performance populations." Performance windows (3/6/12 months) mean a label is not knowable until `observation_date + window` has passed. Including immature or censored observations in a metric silently understates bad rates and corrupts calibration.

## Decision
`fact_performance_label` carries a `maturity_status` field (`mature` / `immature` / `censored`) computed deterministically from `observation_date`, `performance_window_months`, `current_processing_date`, and account closure/charge-off dates. Every monitoring metric pipeline filters to `maturity_status = 'mature'` as a hard, non-overridable WHERE clause, checked by an automated Data Quality Agent rule (`data/DATA_QUALITY_RULES.md`) that fails the pipeline run if violated.

## Consequences
- Positive: directly satisfies the brief's hardest constraint; prevents the single most damaging class of monitoring error (leakage-adjacent bias from immature labels).
- Positive: `censored` is tracked as a distinct status from `immature`, so closed/paid-off accounts are never miscounted as "good" performance.
- Negative: reduces usable sample size for the most recent 1–2 vintages in any monitoring run — this is by design and must be communicated to committee as an expected, not a defect, pattern (`data/TEMPORAL_MODEL.md` §5).

## Alternatives considered
1. **Use all available labels, flag maturity as metadata only** — rejected: metadata that can be silently ignored under time pressure is not a control; the brief requires exclusion, not flagging.
