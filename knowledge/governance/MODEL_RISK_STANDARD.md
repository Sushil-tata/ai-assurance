# Model Risk Standard

## Metadata
- Document Owner: AI Assurance Product and Governance Team
- Version: 1.0
- Effective Date: July 2026
- Review Frequency: Annual
- Status: Draft for Training Demonstration

## Purpose
Provide a practical internal standard for model lifecycle governance and accountability.

## Scope
Applies to all credit risk models referenced in [../models/MODEL_INVENTORY.md](../models/MODEL_INVENTORY.md).

## Lifecycle Expectations
### Proposal
- Define business objective, risk context, and intended use.

### Development
- Document data sources, feature logic, methodology, and limitations.

### Validation
- Independent validation checks design soundness, performance, and implementation.

### Approval
- Human approval required before deployment.

### Deployment
- Controlled release with version traceability.

### Monitoring
- Periodic monitoring across data quality, stability, discrimination, calibration, and applicability.

### Change
- Material change requires governance review and impact evidence.

### Recalibration
- Allowed only with evidence-backed rationale and approval.

### Redevelopment
- Initiated when existing model is not fit for purpose.

### Retirement
- Formal retirement decision with transition plan and residual-risk statement.

## Core Governance Requirements
- Named model ownership and accountability.
- Independent validation function.
- Maintained inventory and status tracking.
- Current documentation and evidence traceability.
- Continuous monitoring and issue management.
- Human accountability for material actions.

## Related Documents
- [INVESTIGATION_GUIDELINES.md](INVESTIGATION_GUIDELINES.md)
- [../policies/MODEL_MONITORING_POLICY.md](../policies/MODEL_MONITORING_POLICY.md)
- [../policies/HUMAN_APPROVAL_POLICY.md](../policies/HUMAN_APPROVAL_POLICY.md)

## Copilot Usage
Use this standard to check lifecycle-stage completeness and accountability before generating governance recommendations.