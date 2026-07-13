# Scenario 005 — MOB Eligibility Breach (B Score Applied Before MOB 6)

**Status:** enterprise/regression test scenario; exercises the applicability-violation path distinct from RAG-based breaches.

## Situation
A feature-engineering pipeline change accidentally removes the `mob >= 6` filter from the B Score scoring population query for one day, causing `CC-BSCORE-v2.1` to score 340 accounts at MOB 3–5.

## Trigger
`data/MONITORING_MART.md` §1.21 applicability check: `COUNT(fact_model_score_event WHERE model_family = 'B' AND mob_at_observation < 6) = 340`, non-zero.

## Handling
Per `agents/MONITORING_AGENT.md` and `policies/ESCALATION_MATRIX.md`, this is `severity = critical` **on detection**, not routed through the normal monthly RAG cycle — it is a structural policy violation (`POL-MMP-5.1`), not a statistical drift signal.

## Finding
"340 accounts were scored by the Behaviour Score model below the MOB 6 eligibility threshold on [date], due to a pipeline change that dropped the eligibility filter. These score events do not represent valid model applications and should not be used operationally." Root cause: `RC-ELIGIBILITY-FILTER-DEFECT` (category: `data_quality`, with a `policy` contributing factor since it's a violation of a specific governed eligibility rule).

## Recommendation
Immediate pipeline fix (already a `feature_fix` type recommendation); the 340 affected score events flagged with a data-quality annotation (not deleted — retained for audit, per retention rules, but excluded from any monitoring population going forward); committee notified same day given `severity = critical`.

## What this scenario tests
That eligibility/applicability rules are enforced as hard, always-on checks independent of the RAG threshold cycle — this is the direct test of ADR-006's sibling rule (label maturity) applied to the *input* side (eligibility) rather than the *output* side (label maturity), and confirms the system doesn't wait for a downstream metric to degrade before catching an eligibility violation.
