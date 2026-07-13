# Agent Risk Policy

**Audience:** Model Risk Committee, AI governance reviewers.
**Purpose:** classify AI Assurance agents under the bank's AI/model risk framework and state the controls that apply.

## 1. Classification

AI Assurance agents are classified as **decision-support tools**, not automated decision systems, under the bank's AI Governance Framework. This classification is justified by, and contingent on, the controls in this repository being implemented as designed:

- No agent takes any customer-impacting action (Principle 1.10).
- No investigation closes without a named human decision (ADR-005).
- Every conclusion is evidence-grounded and citable (Principle 1.4, ADR-008).
- Deterministic calculations are separated from LLM reasoning (Principle 1.1, ADR-003).

If any of the above controls is removed or weakened in a future change, the classification must be re-assessed by the Model Risk Committee before that change ships — this is itself an escalation trigger under `policies/ESCALATION_MATRIX.md`'s spirit even though it isn't a runtime metric.

## 2. Risk tiering of the agents themselves

| Agent | Risk tier | Rationale |
|---|---|---|
| Orchestrator | Low | Pure routing, no domain reasoning, no write access beyond status read |
| Monitoring Agent | Medium | Opens investigations (workflow-triggering), but does not conclude or recommend |
| Data Quality Agent | Medium | Similar profile to Monitoring Agent |
| Governance Agent | Medium | Authoritative on policy interpretation; errors here could mislead downstream reasoning, mitigated by strict grounding requirement |
| Investigation Agent | **High** | Produces the substantive conclusions the whole system exists to generate; highest scrutiny, mandatory critic review |
| Assurance/Critic Agent | **High** | Sole automated gate between Investigation Agent output and human visibility; its own quality is evaluated via the batch rubric process |
| Reporting & Approval Agent | Medium-High | Handles the human-approval workflow; errors here risk misrepresenting evidence to the committee, mitigated by citation-preservation requirement |

## 3. Controls required per tier

| Tier | Controls |
|---|---|
| Low | Execution logging, boundary enforcement |
| Medium | Execution logging, boundary enforcement, output schema validation |
| High | All of the above, plus mandatory adversarial test coverage before any prompt version change ships, plus monthly batch rubric evaluation |

## 4. Change management for agent risk classification

Any proposal to expand an agent's tool access, data domain, or (especially) to remove the human approval gate requires its own ADR and Model Risk Committee sign-off — it is not a routine prompt-engineering change. See `adr/ADR-004-agent-boundaries.md` and `adr/ADR-005-human-approval.md`.
