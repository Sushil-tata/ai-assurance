# Human Approval Policy

**Audience:** Model Risk Committee, Reporting & Approval Agent design owner.
**Companion to:** `architecture/HUMAN_OVERSIGHT.md` (the architectural mechanism); this file is the policy statement (governed, citable, indexed).

## §1 Requirement
No investigation may close, and no recommendation may be actioned, without a decision recorded by a named individual holding appropriate committee or delegated authority, per §2.

## §2 Delegated authority table

| Decision type | Minimum authority |
|---|---|
| `recommendation_type = monitor` or `no_action` | Model Risk Analyst (via `fact_human_approval`, `approval_stage = analyst_review`) may pre-approve for committee ratification, but committee ratification is still required to close per §1 |
| `recommendation_type = investigate_further`, `feature_fix`, `policy_review` | Model Risk Committee decision required |
| `recommendation_type = recalibration`, `retirement` | Model Risk Committee decision required, quorum rules per Committee Charter (external document, referenced not reproduced here) |

## §3 Decision record requirements
Every `fact_committee_decision` row must contain a non-empty `decision_rationale` — a bare "approve" with no rationale text is not policy-compliant and the Reporting & Approval Agent's card UI enforces a minimum rationale length before submission is accepted.

## §4 Request-more-evidence handling
A `request_more_evidence` decision does not close the investigation; it returns `fact_investigation.status` to `open` and is itself logged as a decision event (so the number of evidence-request cycles per investigation is trend-visible, feeding agent-quality evaluation per `agents/AGENT_EVALUATION.md`).

## §5 Override recording
Any committee decision that disagrees with an agent's recommendation is logged to `fact_agent_override` with a categorized reason (`agent_error` / `business_context_agent_lacked` / `policy_judgment_call`) — not merely recorded as a rejection. This distinction matters because only `agent_error` overrides should drive agent-quality corrective action; the other two categories are expected, healthy uses of human judgment.
