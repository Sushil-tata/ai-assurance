# C Score

## Metadata
- Document Owner: AI Assurance Product and Governance Team
- Version: 1.0
- Effective Date: July 2026
- Review Frequency: Monthly
- Status: Draft for Training Demonstration

## Purpose
Define bucket-specific collections monitoring rules for C Score models.

## Scope
Limited to Bucket 0 and Bucket 1 monitoring for Credit Card, Speedy Cash, and Speedy Loan.

## Family Definition
- C Score is a delinquency transition score family.
- Monitoring is bucket-specific and must not mix Bucket 0 with Bucket 1 populations.
- This runtime scope does not expand beyond Bucket 1.

## Bucket Definitions
### Bucket 0
Current or near-current accounts used to monitor early deterioration risk.

### Bucket 1
Early delinquent accounts used to monitor worsening risk and cure likelihood.

## Outcome States
- Cure: transition to materially improved repayment status.
- Stay: remains in same bucket over monitoring interval.
- Roll-forward: transition to worse bucket.
- NPL: non-performing status per internal credit policy classification.
- Charge-off: write-off event based on internal accounting and policy criteria.
- Restructuring: payment-term modification due to distress handling.

## Observation Populations
- Bucket 0 model evaluates only accounts observed in Bucket 0 at observation time.
- Bucket 1 model evaluates only accounts observed in Bucket 1 at observation time.
- Cross-bucket leakage or pooled labeling should be treated as data design error.

## Monitoring Focus
- Transition rates by bucket and product.
- Cure and redefault behavior after interventions.
- Roll-forward concentration by segment and vintage.
- Data quality checks for delinquency status coding and timing.

## Related Documents
- [MODEL_INVENTORY.md](MODEL_INVENTORY.md)
- [B_SCORE.md](B_SCORE.md)
- [../policies/MODEL_MONITORING_POLICY.md](../policies/MODEL_MONITORING_POLICY.md)

## Copilot Usage
Use this file when interpreting collections transition signals and when drafting bucket-specific findings and recommendations.