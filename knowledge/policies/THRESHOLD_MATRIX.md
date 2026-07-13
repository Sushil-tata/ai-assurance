# Threshold Matrix

## Metadata
- Document Owner: AI Assurance Product and Governance Team
- Version: 1.0
- Effective Date: July 2026
- Review Frequency: Quarterly
- Status: Draft for Training Demonstration

## Purpose
Provide illustrative Green/Amber/Red thresholds for monitoring triage in AI Assurance.

## Scope
Applies to runtime interpretation of deterministic monitoring outputs.

## Important Note
All thresholds below are synthetic demonstration thresholds and are not CardX production policy.

## Synthetic Thresholds
| Signal | Green | Amber | Red |
|---|---|---|---|
| PSI | < 0.10 | 0.10 to < 0.25 | >= 0.25 |
| Gini decline versus reference | < 3 percentage points | 3 to < 8 percentage points | >= 8 percentage points |
| O/E ratio | 0.90 to 1.10 | 0.80 to < 0.90 or > 1.10 to 1.20 | < 0.80 or > 1.20 |
| Missingness increase in key features | < 2 percentage points | 2 to < 5 percentage points | >= 5 percentage points |
| Sample-size sufficiency (eligible records) | >= 5,000 | 2,000 to < 5,000 | < 2,000 |
| Model applicability violations | 0% | > 0% to <= 1% | > 1% |
| Model-version mismatch | none | one isolated run corrected same cycle | persistent or unresolved mismatch |
| Validation expiry | valid for > 90 days | <= 90 days to expiry | expired |

## Conflicting Signal Handling
- Red in any applicability or implementation control overrides metric Greens.
- Multiple Amber signals across independent dimensions should be treated as consolidated Red for escalation review.
- If stability is Red but discrimination is Green, require investigation before remediation decision.
- Missingness Red with metric instability should prioritize data remediation first.

## Key Terms
- Isolated run: single incident with immediate correction and traceable evidence.
- Persistent mismatch: repeated or unresolved implementation inconsistency.

## Related Documents
- [MODEL_MONITORING_POLICY.md](MODEL_MONITORING_POLICY.md)
- [ESCALATION_MATRIX.md](ESCALATION_MATRIX.md)
- [../governance/EVIDENCE_GUIDELINES.md](../governance/EVIDENCE_GUIDELINES.md)

## Copilot Usage
Use this matrix for deterministic severity tagging and escalation recommendation framing, while explicitly labeling values as synthetic thresholds.