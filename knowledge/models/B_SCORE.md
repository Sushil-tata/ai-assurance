# B Score

## Metadata
- Document Owner: AI Assurance Product and Governance Team
- Version: 1.0
- Effective Date: July 2026
- Review Frequency: Monthly
- Status: Draft for Training Demonstration

## Purpose
Define the monthly post-booking behavior score family and its monitoring interpretation rules.

## Scope
Applies to B Score models for Credit Card, Speedy Cash, and Speedy Loan in account-month monitoring.

## Family Definition
- B Score is a monthly post-booking score.
- B Score applicability is only when MOB >= 6.
- Observation unit is account-month.
- Any scoring event with MOB < 6 is an applicability violation.

## Monthly Rescoring
- Each eligible account is rescored monthly.
- Monitoring compares current month versus reference periods and recent trends.
- Investigation should separate scoring-population change from pure model drift.

## Forward Performance Windows
- Forward 3-month: earliest performance readout after observation month.
- Forward 6-month: intermediate readout.
- Forward 12-month: long-horizon readout.
- Immature windows cannot be used for outcome conclusions.

## Eligibility and Applicability Controls
- Enforce MOB >= 6 before metric generation and narrative interpretation.
- Record count and proportion of ineligible scored records.
- Route breaches through applicability controls in [../policies/THRESHOLD_MATRIX.md](../policies/THRESHOLD_MATRIX.md).

## Revolving and Term-Loan Distinctions
### Revolving Products (Credit Card, Speedy Cash)
- Utilization dynamics and payment ratio changes can materially affect score distribution.
- Campaign-driven thin-file mix can alter stability metrics without immediate discrimination collapse.

### Term Loan Product (Speedy Loan)
- Installment adherence and contractual schedule progression are dominant behavioral drivers.
- Utilization-ratio style interpretations are not meaningful for term loans.

## Monitoring Interpretation Guide
### Data Quality
- Validate feature missingness, source refresh timing, and schema compatibility.

### Stability
- Use PSI and feature-level CSI to detect shift.

### Discrimination
- Track Gini, KS, and AUC trend by product and key segments.

### Calibration
- Evaluate O/E ratio and segment-level calibration drift.

### Segment Performance
- Analyze by vintage, channel, thin-file segment, and policy cohorts.

### Operational Implementation
- Confirm correct model version, cutoff logic, and scoring population filters.

## June 2026 MVP Scenario Signals
- Primary shift: thin-file mix increase after campaign.
- Contributing shift: bureau feature missingness increase from upstream issue.
- Population change: policy change altered scored population composition.
- Expected outcome: combined stability and calibration stress with potentially mixed discrimination signals.

## Key Terms
- Account-month: account-level observation for a specific month.
- Applicability violation: use of B Score outside MOB >= 6 rule.

## Related Documents
- [MODEL_INVENTORY.md](MODEL_INVENTORY.md)
- [../business/PRODUCTS.md](../business/PRODUCTS.md)
- [../policies/MODEL_MONITORING_POLICY.md](../policies/MODEL_MONITORING_POLICY.md)
- [../governance/INVESTIGATION_GUIDELINES.md](../governance/INVESTIGATION_GUIDELINES.md)

## Copilot Usage
Use this as the primary runtime model-family guide for monthly monitoring narratives, applicability checks, and investigation hypothesis framing.