# Escalation Matrix

## Metadata
- Document Owner: AI Assurance Product and Governance Team
- Version: 1.0
- Effective Date: July 2026
- Review Frequency: Quarterly
- Status: Draft for Training Demonstration

## Purpose
Define Green, Amber, and Red escalation paths, response times, and required evidence.

## Scope
Applies to model monitoring, investigation findings, and governance exceptions.

## Roles
- Model Owner
- Model Risk
- Data Owner
- Technology Owner
- Business Owner
- Governance Committee

## Escalation Levels
| Level | Typical Condition | Response Time | Required Evidence | Required Participants |
|---|---|---|---|---|
| Green | Normal variation within policy limits | 10 business days | Monitoring summary, metric extract, policy check output | Model Owner |
| Amber | Material warning or repeated minor breach | 5 business days | Investigation note, evidence ledger, impact assessment | Model Owner, Model Risk, Data Owner |
| Red | Severe breach, applicability failure, implementation risk, or governance expiry | 2 business days | Full investigation pack, root-cause evidence, remediation recommendation, approval request | Model Owner, Model Risk, Data Owner, Technology Owner, Business Owner, Governance Committee |

## Automatic Escalation Conditions
- Any Red threshold in applicability controls.
- Persistent model-version mismatch.
- Validation expired.
- Repeated Amber over three consecutive cycles.
- Contradictory evidence that invalidates prior Green or Amber conclusion.

## Evidence Minimum
- Evidence IDs and source references.
- Deterministic metric outputs.
- Policy version used for evaluation.
- Trace of tool call or workflow run that produced key values.

## Related Documents
- [THRESHOLD_MATRIX.md](THRESHOLD_MATRIX.md)
- [REMEDIATION_PLAYBOOK.md](REMEDIATION_PLAYBOOK.md)
- [../governance/COMMITTEE_GUIDELINES.md](../governance/COMMITTEE_GUIDELINES.md)

## Copilot Usage
Use this matrix to route stakeholders, set urgency in generated summaries, and confirm minimum evidence before escalation recommendation.