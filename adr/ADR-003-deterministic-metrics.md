# ADR-003: Deterministic Metric Calculation Outside the LLM

**Status:** Accepted
**Date:** 2026-07-13
**Deciders:** Model Risk Architect, Principal Enterprise Data Architect

## Context
Monitoring metrics (PSI, Gini, KS, calibration, bad rate) must be numerically exact, reproducible, and defensible to a model risk committee and, potentially, a regulator. LLM-based arithmetic over raw rows is not reliable at this precision or scale, and is not reproducible run-to-run.

## Decision
All monitoring metrics are computed by versioned SQL/Azure Function pipelines and written to `fact_monitoring_metric` / `fact_segment_monitoring_metric` with a `calculation_version` field. Agents access these values only via a read-only tool call (`get_monitoring_metric`); no agent instruction ever asks an LLM to compute or estimate a statistical value.

## Consequences
- Positive: every number in a committee report is re-runnable and independently verifiable outside the LLM entirely.
- Positive: `calculation_version` allows a metric definition change (e.g., PSI binning method) to be tracked and old values to remain interpretable in their original context.
- Negative: any new metric requires a pipeline change, not just a prompt change — slower to add ad hoc metrics. Accepted trade-off given the governance context.

## Alternatives considered
1. **LLM computes metrics from raw data in-context** — rejected outright: unverifiable, non-reproducible, and directly contradicts Principle 1.1.
2. **Hybrid: LLM computes "quick" metrics, pipeline computes "official" metrics** — rejected: creates two sources of truth with no clear precedence rule; confuses committee reviewers.
