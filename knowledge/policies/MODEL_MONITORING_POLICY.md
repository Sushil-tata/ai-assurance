# Model Monitoring Policy

## Metadata
- Document Owner: AI Assurance Product and Governance Team
- Version: 1.0
- Effective Date: July 2026
- Review Frequency: Monthly
- Status: Draft for Training Demonstration

## Purpose
Define minimum monitoring objectives, review areas, and governance constraints for AI Assurance.

## Scope
Applies to A Score, B Score, and C Score monitoring across Credit Card, Speedy Cash, and Speedy Loan.

## Monitoring Objectives
- Detect material deterioration, instability, or implementation error.
- Separate data quality issues from model logic issues.
- Ensure model usage remains within validated scope and eligibility rules.
- Route material actions to human approval.

## Minimum Review Areas
- Data quality.
- Applicability.
- Stability.
- Discrimination.
- Calibration.
- Segment performance.
- Operational implementation.
- Governance status.

## Operating Rules
- Deterministic metrics must be calculated outside the LLM.
- Agents may interpret deterministic outputs but cannot fabricate metric values.
- Immature performance windows cannot be used for outcome monitoring conclusions.
- Monitoring output must distinguish observed fact, inference, and recommendation.
- Agents recommend actions; humans approve material actions.

## Applicability Controls
- B Score must be applied only when MOB >= 6.
- A Score is acquisition-only and must not be interpreted as post-booking monthly score.
- C Score monitoring must stay bucket-specific.

## Governance Linkage
- Threshold interpretation: [THRESHOLD_MATRIX.md](THRESHOLD_MATRIX.md)
- Escalation path: [ESCALATION_MATRIX.md](ESCALATION_MATRIX.md)
- Remediation options: [REMEDIATION_PLAYBOOK.md](REMEDIATION_PLAYBOOK.md)
- Human decisions: [HUMAN_APPROVAL_POLICY.md](HUMAN_APPROVAL_POLICY.md)

## Copilot Usage
Use this file as the base policy frame for monitoring narratives, policy checks, and recommendation gating.