# A Score

## Metadata
- Document Owner: AI Assurance Product and Governance Team
- Version: 1.0
- Effective Date: July 2026
- Review Frequency: Quarterly
- Status: Draft for Training Demonstration

## Purpose
Define the acquisition-only model family for AI Assurance monitoring and governance workflows.

## Scope
Applies to A Score models for Credit Card, Speedy Cash, and Speedy Loan at application stage only.

## Family Definition
- A Score is acquisition-only.
- Observation date is the application date.
- Observation unit is approved application.
- Scores are not re-used as post-booking monthly behavior scores.

## Forward Performance Windows
- 3-month window: early performance signal.
- 6-month window: intermediate performance signal.
- 12-month window: more stable performance signal.
- Outcome monitoring uses only windows that are mature.

## Label Maturity Rules
- Mature label: full target window elapsed and outcome finalized per policy.
- Immature label: target window has not fully elapsed.
- Censored label: outcome interrupted or incomplete for non-performance reasons.

## Leakage Controls
- Exclude post-application features from development and monitoring datasets.
- Lock feature snapshot at application date.
- Separate booking decision logic from outcome labels.
- Flag any detected post-observation data contamination as a governance issue.

## Key Terms
- Acquisition-only: usable only at origination stage.
- Observation date: point-in-time date used for feature capture and eligibility.

## Related Documents
- [MODEL_INVENTORY.md](MODEL_INVENTORY.md)
- [B_SCORE.md](B_SCORE.md)
- [../policies/MODEL_MONITORING_POLICY.md](../policies/MODEL_MONITORING_POLICY.md)

## Copilot Usage
Use this document when interpreting application-stage monitoring signals and when checking that no post-booking leakage assumptions were introduced.