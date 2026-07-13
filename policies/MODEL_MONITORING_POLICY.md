# Model Monitoring Policy (MMP) — v4

**Status:** Source document represented in `dim_policy_document` (`document_code = MMP`) and `dim_policy_clause`. This markdown file is the human-readable working copy; the governed, citable version of record is the SharePoint PDF indexed into Azure AI Search (`architecture/SYSTEM_ARCHITECTURE.md`).

## §1 Purpose and scope
This policy governs monitoring of all production credit risk models (A/B/C Score families) across Credit Card, Speedy Cash and Speedy Loan. It defines monitoring frequency, metric requirements, RAG thresholds, and the investigation obligation upon breach.

## §2 Monitoring frequency
§2.1 Behaviour Score (B) models are monitored monthly, aligned to the monthly scoring cycle.
§2.2 Application Score (A) models are monitored monthly on a rolling application cohort basis.
§2.3 Collections Score (C) models are monitored monthly per delinquency bucket.

## §3 Population Stability Index (PSI)
§3.1 PSI is computed on the score distribution for every production model at every monitoring run.
§3.2 A PSI value between the Green and Amber bounds defined in the applicable `dim_governance_threshold` row shall be classified Amber and shall trigger an investigation within 5 business days of the monitoring run date. *(This is `POL-MMP-3.2`, cited throughout `scenarios/SCENARIO-001-psi-breach.md`.)*
§3.3 A PSI value above the Amber bound shall be classified Red and requires same-week notification to the Model Risk Committee chair in addition to the standard investigation.

## §4 Performance metrics and label maturity
§4.1 No performance-based metric (bad rate, Gini, KS, calibration) may be computed using an immature or censored performance-label population. *(Directly implements ADR-006.)*
§4.2 Minimum sample-size requirements per metric are defined in `data/MONITORING_MART.md` and enforced via `statistical_reliability_flag`.

## §5 Model eligibility and applicability
§5.1 Behaviour Score models shall not be applied to accounts with fewer than 6 months on book. Any such application is a Red-severity applicability violation requiring immediate investigation, independent of the standard RAG cycle.

## §6 Investigation and escalation
§6.1 Every Amber or Red monitoring breach shall open a structured investigation per the model defined in `data/GOVERNANCE_SCHEMA.md`.
§6.2 Escalation thresholds and routing are defined in `policies/ESCALATION_MATRIX.md`.

## §7 Threshold governance
§7.1 All monitoring thresholds are maintained as versioned, approved records in the governance threshold registry. No threshold may be applied that lacks a named approver and effective date. *(Implements ADR-007.)*

## §8 Roles and accountability
§8.1 Retail Risk Analytics owns day-to-day monitoring execution.
§8.2 Model Risk Committee owns threshold approval and investigation closure decisions.
§8.3 No automated system may close an investigation without a named human decision-maker. *(Implements ADR-005.)*

## Version history
| Version | Effective date | Change |
|---|---|---|
| v3 | 2024-06-01 | Prior version |
| v4 | 2025-01-01 | Added AI Assurance investigation obligation (§6), clarified label maturity rule (§4.1) |
