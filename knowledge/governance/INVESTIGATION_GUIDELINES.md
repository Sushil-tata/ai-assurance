# Investigation Guidelines

## Metadata
- Document Owner: AI Assurance Product and Governance Team
- Version: 1.0
- Effective Date: July 2026
- Review Frequency: Quarterly
- Status: Draft for Training Demonstration

## Purpose
Define a consistent evidence-led investigation workflow for monitoring signals.

## Scope
Applies to all model families, with primary runtime focus on June 2026 CC Behaviour Score investigation.

## Standard Workflow
trigger -> scope -> evidence collection -> hypotheses -> testing -> counter-evidence -> finding -> root cause -> recommendation -> assurance review -> human decision

## Workflow Controls
- Keep facts separate from inference.
- Track each claim to evidence ID.
- Evaluate contradictory evidence before final finding.
- Use policy and threshold references explicitly.

## Worked Scenario: June 2026 CC Behaviour Score
### Trigger
Monthly monitoring raised stability and calibration concern for CC Behaviour Score.

### Scope
- Product: Credit Card.
- Model: B Score.
- Window: June 2026 cycle.

### Evidence Collection
- Population mix by thin-file status.
- Bureau feature missingness trend.
- Policy cohort flags before and after policy change.
- Deterministic metrics for PSI, Gini, and O/E.

### Hypotheses
- H1: Thin-file campaign mix is the primary cause.
- H2: Bureau missingness increase is a contributing factor.
- H3: Policy change altered scored population and amplified metrics.

### Counter-Evidence
- Check whether non-thin-file segments showed independent deterioration.
- Check whether missingness correction simulations reduce instability.
- Check whether policy cohort normalization narrows observed shift.

### Investigation Outcome Framing
- Primary cause: higher thin-file mix after campaign.
- Contributing factor: increased bureau feature missingness from upstream issue.
- Unrelated signal: isolated non-material segment volatility with no persistent effect.

### Recommendation Pattern
- Data remediation and enhanced monitoring first.
- Escalate for human approval if impact remains material after correction.

## Related Documents
- [EVIDENCE_GUIDELINES.md](EVIDENCE_GUIDELINES.md)
- [../models/B_SCORE.md](../models/B_SCORE.md)
- [../policies/REMEDIATION_PLAYBOOK.md](../policies/REMEDIATION_PLAYBOOK.md)

## Copilot Usage
Use this file to build investigation narratives in the required order and to enforce primary cause, contributing factor, and unrelated signal separation.