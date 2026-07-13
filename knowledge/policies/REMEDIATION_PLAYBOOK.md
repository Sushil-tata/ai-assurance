# Remediation Playbook

## Metadata
- Document Owner: AI Assurance Product and Governance Team
- Version: 1.0
- Effective Date: July 2026
- Review Frequency: Quarterly
- Status: Draft for Training Demonstration

## Purpose
Define permitted remediation actions with clear triggers, owners, evidence, approval requirements, and closure criteria.

## Scope
Applies after investigation and policy classification for A Score, B Score, and C Score monitoring events.

## Action Matrix
| Action | Typical Trigger | Owner | Evidence Required | Human Approval | Closure Criteria |
|---|---|---|---|---|---|
| No action | Green outcome with no material risk | Model Owner | Monitoring summary and policy pass | No | One full cycle remains within tolerance |
| Enhanced monitoring | Amber trend without confirmed root-cause severity | Model Owner | Trend analysis and enhanced plan | No | Two consecutive cycles show stabilization |
| Data remediation | Confirmed data quality defect | Data Owner | Defect evidence, fix validation, backfill check | Yes if material impact | Data quality metric returns to Green |
| Pipeline correction | Scoring or transformation implementation error | Technology Owner | Incident log, correction proof, rerun evidence | Yes | Version and output alignment restored |
| Applicability correction | B Score used with MOB < 6 or out-of-scope usage | Model Owner | Applicability breach quantification and control fix | Yes | Zero violations for next cycle |
| Cutoff or strategy review | Segment-level risk shift impacting decisions | Business Owner | Impact simulation and governance note | Yes | Committee-approved strategy change recorded |
| Recalibration | Persistent calibration drift with stable implementation | Model Risk | Calibration evidence and challenger assessment | Yes | Approved recalibration and post-change validation |
| Challenger review | Material uncertainty in current model adequacy | Model Risk | Challenger performance and stability comparison | Yes | Decision on champion/challenger path |
| Redevelopment | Structural deterioration or persistent model inadequacy | Model Owner | Redevelopment plan and timeline | Yes | New model validated and approved |
| Temporary use restriction | Acute risk requiring immediate containment | Business Owner | Risk statement and control measures | Yes | Restriction lifted by committee decision |
| Retirement review | Model no longer fit, needed, or compliant | Model Risk | Retirement rationale and transition plan | Yes | Formal retirement decision logged |

## Key Terms
- Material impact: expected business or governance consequence requiring human sign-off.
- Closure criteria: objective endpoint required to close remediation action.

## Related Documents
- [MODEL_MONITORING_POLICY.md](MODEL_MONITORING_POLICY.md)
- [HUMAN_APPROVAL_POLICY.md](HUMAN_APPROVAL_POLICY.md)
- [../governance/EVIDENCE_GUIDELINES.md](../governance/EVIDENCE_GUIDELINES.md)

## Copilot Usage
Use this playbook to draft action recommendations and verify that each recommendation includes owner, evidence, approval gate, and closure conditions.