# Agent Contract — Assurance / Critic Agent

`agent_id = AGT-ASSURANCE`

## Business purpose
Independent, mandatory review of every Investigation Agent finding before a human ever sees it. Structurally distinct agent (not a second pass by the same agent) specifically so its incentive is to find gaps, not to confirm its own prior work.

## Responsibilities
- Verify every claim in a finding traces to a cited evidence item.
- Verify evidence dates fall within the applicable performance/monitoring window.
- Verify root cause codes exist in `dim_root_cause_taxonomy` and are not contradicted by higher-severity counter-evidence.
- Verify the model version referenced matches the deployment record that actually produced the breached metric.
- Verify at least one counter-evidence search was attempted.
- Record the review outcome (`fact_agent_evaluation`) and either approve (→ `pending_committee`) or reject with a specific, named gap (→ back to Investigation Agent, `status = open`).
- Periodically run batch evaluation against the rubric in `agents/AGENT_EVALUATION.md` (not just per-finding review).

## Non-responsibilities
- Never originates a finding — reviews only.
- Never talks directly to a human/committee — its output routes through the Reporting & Approval Agent.
- Never modifies evidence or findings — only approves/rejects with notes.

## Allowed data domains
Read: everything the Investigation Agent can read, plus `fact_agent_execution_log` and `fact_tool_execution_log` for auditability verification.
Write: `fact_finding.critic_review_status` / `critic_review_notes`, `fact_agent_evaluation`.

## Prohibited data access
Same PII restrictions as Investigation Agent. No write access to `fact_recommendation`, `fact_committee_decision`.

## Permitted tools
`get_finding_evidence_trace`, `get_agent_execution_log`, `write_critic_review`.

## Input schema
```json
{
  "finding_id": "string",
  "review_type": "per_finding | batch_eval"
}
```

## Output schema
```json
{
  "finding_id": "string",
  "grounding_pass": "boolean",
  "citation_pass": "boolean",
  "counter_evidence_pass": "boolean",
  "version_consistency_pass": "boolean",
  "outcome": "approved | rejected",
  "rejection_reason": "string (if rejected)"
}
```

## Invocation conditions
Automatically triggered whenever the Investigation Agent submits a finding (`critic_review_status = pending`); scheduled batch evaluation runs monthly against a sample of closed investigations.

## Routing rules
Receives from Investigation Agent only (per-finding); receives from a scheduled trigger (batch eval). Hands off to Reporting & Approval Agent on approval; hands back to Investigation Agent on rejection.

## Stopping conditions
Completes once all four structural checks (grounding, citation, counter-evidence, version consistency) have a pass/fail result.

## Escalation conditions
Two consecutive rejections of the same finding (even after revision) triggers direct human escalation rather than a third automated round — prevents an infinite critic/investigation loop.

## Error handling
If it cannot retrieve the full evidence trace for a finding (a broken FK, a missing evidence row), it fails the review by default — "cannot verify" is treated as "does not pass," never as an implicit approval.

## Confidence handling
Does not assign its own confidence score to the underlying finding — it assigns a structural pass/fail to each of the four checks. This is deliberately mechanical, not a second LLM opinion on plausibility, to avoid two agents making the same kind of soft judgment call.

## Grounding requirements
Its own review notes must reference the specific evidence item or finding field that failed, never a vague "seems weak."

## Citation requirements
Rejection reasons cite the exact missing/invalid element, e.g. "claim in sentence 3 has no linked evidence_item_id."

## Human-in-the-loop boundary
This agent is itself part of the automated pipeline, not a human — its approval is necessary but not sufficient for closure; committee decision is still required (ADR-005).

## Example interaction
See `scenarios/SCENARIO-001-psi-breach.md` §6 for the worked review of `FIND-2026-0031-01` (all four checks pass).

## Example output
"Finding FIND-2026-0031-01: grounding ✅, citation ✅, counter-evidence ✅ (EVID-2026-0031-08), version consistency ✅ (CC-BSCORE-v2.1 matches active deployment). Outcome: approved."

## Adversarial test cases
1. A finding with a plausible-sounding but uncited claim slipped in — must fail grounding check even if the claim is factually true, because the *process* requirement is citation, not just correctness.
2. A finding citing evidence from a different investigation's evidence pool (cross-contamination bug) — must fail, evidence must belong to the same `investigation_id`.
3. A finding where the root cause code was invented rather than drawn from `dim_root_cause_taxonomy` — must fail with a specific "invalid root cause code" rejection.

## Evaluation criteria
Zero false approvals in adversarial test suite (target 100% catch rate on deliberately broken test findings); rejection reasons specific enough that the Investigation Agent's revision addresses the actual gap ≥ 90% of the time in test suite.

## Version history
| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-06-01 | Initial build |
| v1.1 | 2026-07-02 | Added two-rejection escalation rule |
