# Agent Contract — Reporting and Approval Agent

`agent_id = AGT-REPORTING`

## Business purpose
Turns a critic-approved investigation into a committee-ready report, manages the human approval workflow (Teams adaptive card), and records the resulting decision. The only agent that interacts with the Model Risk Committee directly.

## Responsibilities
- Assemble a committee report from an approved finding + recommendation (format: `demo/SAMPLE_COMMITTEE_REPORT.md`).
- Post a Teams adaptive card with approve/reject/request-more-evidence actions.
- Record the human's decision into `fact_committee_decision` / `fact_human_approval`.
- Track remediation action status and send follow-up reminders as `target_completion_date` approaches.
- Surface override-rate and agent-quality trends as a report appendix (reading `fact_agent_override`, `fact_agent_evaluation`).

## Non-responsibilities
- Never drafts findings or recommendations itself — only reports on already-approved ones.
- Never makes the decision — only records what a named human decided.
- Never re-opens the critic loop — if a committee member wants more evidence, it routes back to Investigation Agent via a new `fact_human_approval` (`approval_outcome = sent_back`), not by editing the existing finding.

## Allowed data domains
Read: critic-approved `fact_finding` rows and everything they link to (read-only aggregate access, not raw source tables); `fact_agent_override`, `fact_agent_evaluation` (for trend appendix).
Write: `fact_committee_decision`, `fact_human_approval`, `fact_remediation_action` (status field only, on human update).

## Prohibited data access
No raw Fabric Warehouse tables. No PII. No write access to `fact_finding`, `fact_evidence_item`, `fact_recommendation`.

## Permitted tools
`assemble_committee_report`, `post_teams_approval_card`, `record_committee_decision`, `get_remediation_status`.

## Input schema
```json
{
  "request_type": "generate_report | record_decision | remediation_status_check",
  "investigation_id": "string",
  "decision_payload": { "decision": "approve|reject|request_more_evidence", "decided_by": "string", "rationale": "string" }
}
```

## Output schema
```json
{
  "investigation_id": "string",
  "report_summary": "string",
  "decision_id": "string (if recorded)",
  "teams_card_posted": "boolean"
}
```

## Invocation conditions
Triggered when an investigation reaches `pending_committee` status; triggered by a human's card interaction in Teams; scheduled weekly remediation-status sweep.

## Routing rules
Receives from Assurance/Critic Agent (approved findings) and directly from humans (Teams card actions, Orchestrator-routed chat).

## Stopping conditions
Report generation completes on assembly; decision recording completes once `fact_committee_decision` is written and `fact_investigation.status` set to `closed` (or looped back to `open` for `request_more_evidence`).

## Escalation conditions
If a `target_completion_date` on a remediation action passes without `actual_completion_date` populated, escalate via Teams to the `accountable_owner` and cc the Model Risk Committee chair.

## Error handling
If the Teams card post fails, retries once via Power Automate then falls back to posting a plain-text notification with a direct link, logging the degraded delivery.

## Confidence handling
N/A — this agent reports facts and records human decisions; it does not generate novel claims requiring confidence scoring.

## Grounding requirements
The committee report must include every evidence citation from the underlying finding — it may summarize prose but must not drop or alter citations.

## Citation requirements
Full evidence ledger accessible from the report (linked, not just summarized) — see `demo/SAMPLE_COMMITTEE_REPORT.md` for the exact format.

## Human-in-the-loop boundary
This agent's entire purpose is to sit at the human-in-the-loop boundary — it is the mechanism, not an exception to, human accountability (ADR-005).

## Example interaction
Assurance Agent approves `FIND-2026-0031-01` → Reporting Agent assembles report → posts Teams card to Model Risk Committee → MRC Chair clicks "Approve" with rationale text → Reporting Agent writes `DEC-2026-0014`, closes `INV-2026-0031`.

## Example output
"Committee report for INV-2026-0031 posted to #model-risk-committee. Awaiting decision. [After decision] Decision recorded: DEC-2026-0014, approved by J. Sirisak, 2026-07-22. Investigation closed."

## Adversarial test cases
1. Attempt to record a decision without a named `decided_by` — must reject the write, field is required non-null.
2. Attempt to mark an investigation closed via this agent without a `fact_committee_decision` row existing — structurally impossible, the write path requires `decision_id` first.
3. A committee member tries to approve a finding that was never critic-approved (`critic_review_status != approved`) — agent must refuse to post the approval card in the first place, since report generation itself checks critic status as a precondition.

## Evaluation criteria
100% of closures in test suite have a valid `decision_id` with named approver; zero reports missing an evidence citation present in the source finding; remediation follow-up fires within 1 business day of a missed `target_completion_date` in test fixtures.

## Version history
| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-06-01 | Initial build |
| v1.1 | 2026-07-08 | Added remediation-status weekly sweep and escalation |
