# Human Oversight

**Audience:** Model Risk Committee, compliance, Copilot Studio builders implementing the approval card.

## 1. The human-in-the-loop boundary, precisely stated

No investigation may transition to `status = closed` and no recommendation may transition to `status = actioned` without a row in `fact_committee_decision` or `fact_human_approval` naming a real person (Entra identity), a decision (`approve` / `reject` / `request_more_evidence`), and a timestamp. This is enforced structurally: the Reporting & Approval Agent's only write path to `fact_investigation.status = closed` requires a non-null `decision_id` foreign key — there is no code path that sets closure without it.

## 2. Where humans sit in the flow

| Stage | Human role | What they see | What they can do |
|---|---|---|---|
| Monitoring breach detected | None required yet | Optional Teams notification | Nothing mandatory |
| Investigation opened | Model Risk Analyst (informed) | Teams notification with investigation summary | Can ask the Orchestrator questions at any time; not required to act |
| Finding + recommendation drafted, critic-approved | Model Risk Analyst (reviewer) | Full evidence ledger, finding, root cause, recommendation via Reporting & Approval Agent | Can request more evidence (sends back to Investigation Agent), or forward to committee |
| Committee review | Model Risk Committee (decision authority) | Committee report (`demo/SAMPLE_COMMITTEE_REPORT.md` format) with full evidence trace | Approve remediation, reject finding, request further investigation — recorded in `fact_committee_decision` |
| Remediation | Accountable business owner (named in recommendation) | Recommendation with owner assignment | Executes remediation *outside* AI Assurance; records completion evidence back into `fact_remediation_action` |

## 3. Escalation triggers that force human involvement

See `policies/ESCALATION_MATRIX.md` for the full table. Structural triggers that cannot be bypassed by any agent:
- Any Red RAG status on a materiality-tier-1 model.
- Any finding whose root cause taxonomy code is `policy_change` or `model_version_mismatch` (these have direct governance implications beyond a single metric).
- Any investigation where the Assurance/Critic Agent has rejected the same finding twice.
- Any recommendation whose action type is `recalibration` or `retirement` (model lifecycle changes always require committee-level sign-off, never just an analyst).

## 4. Why this is not "human rubber-stamping"

The committee is given the full evidence ledger, not a summarized conclusion — every citation is a live link to a Dataverse row, so a reviewer can drill into the exact metric value, the exact policy clause, and the exact tool call that produced it. The Assurance/Critic Agent's job (§3 of `architecture/AGENT_ARCHITECTURE.md`) is specifically to make sure the human is never asked to approve an ungrounded claim — the human's judgment is spent on materiality and business context, not on re-deriving whether the PSI calculation was correct.

## 5. Accountability record retention

`fact_committee_decision` and `fact_human_approval` are retained indefinitely as governance records (not subject to the 2-year rolling retention applied to raw agent execution logs), because they are the permanent record of who approved what, and are the artifact a regulator or internal audit would request first.
