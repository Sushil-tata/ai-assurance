# Scenario 006 — Expired Model Validation (Governance Lifecycle Breach)

**Status:** enterprise/regression test scenario; exercises the Governance Agent and lifecycle-stage monitoring, distinct from statistical monitoring entirely.

## Situation
`CC-BSCORE-v2.1`'s `next_review_date` (2026-09-15, per `data/MODEL_INVENTORY_SCHEMA.md`) passes without a revalidation being completed or a committee-approved extension being granted.

## Trigger
A scheduled governance check (analogous to the monitoring cycle, but on `dim_model_version.next_review_date` rather than a statistical metric) run by the Governance Agent finds `next_review_date < current_date` with no superseding validated version and no extension record.

## Finding
"CC Behaviour Score v2.1's scheduled model risk review date has passed without revalidation or a documented extension. Continued production use without a current validation is a governance policy breach independent of the model's statistical performance." Root cause: `RC-VALIDATION-EXPIRED` (category: `policy`).

## Recommendation
`recommendation_type = policy_review` — escalate to Model Validation Team to schedule revalidation immediately; committee to decide whether continued production use is permitted pending revalidation or whether the model should be temporarily suspended (a `retirement`-adjacent decision requiring committee authority, per `policies/HUMAN_APPROVAL_POLICY.md` §2).

## What this scenario tests
That AI Assurance's governance layer monitors the *model lifecycle itself* (validation currency), not only statistical performance — a model can be performing perfectly well on every PSI/Gini/KS metric and still be in governance breach because its validation lapsed. This is the clearest illustration of why `data/MODEL_INVENTORY_SCHEMA.md`'s lifecycle-stage tracking exists as a first-class structure rather than an afterthought field.
