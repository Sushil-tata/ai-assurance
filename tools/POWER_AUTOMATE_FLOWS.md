# Power Automate Flows

**Audience:** Copilot Studio / Power Platform builders.

## `Flow-01: Monthly Monitoring Trigger`
**Trigger:** Recurrence, monthly, 1st business day, 06:00.
**Action:** Invokes Monitoring Agent (`run_type = scheduled`) via Copilot Studio API trigger.
**Failure handling:** If the agent call fails, retries twice with exponential backoff, then posts a Teams alert to the engineering channel (not the Model Risk Committee channel — this is an operational failure, not a governance event).

## `Flow-02: Investigation Created Notification`
**Trigger:** Dataverse row-create on `fact_investigation`.
**Action:** Posts a Teams message to `#model-risk-analytics` summarizing `trigger_description`; invokes Investigation Agent with the new `investigation_id`.
**Failure handling:** Notification failure logged but does not block the Investigation Agent invocation (notification is best-effort, investigation processing is not).

## `Flow-03: Finding Ready for Critic Review`
**Trigger:** Dataverse row-create on `fact_finding` where `critic_review_status = pending`.
**Action:** Invokes Assurance/Critic Agent.

## `Flow-04: Finding Critic-Approved`
**Trigger:** Dataverse row-update on `fact_finding` where `critic_review_status` changes to `approved`.
**Action:** Invokes Reporting & Approval Agent to assemble and post the committee report/approval card.

## `Flow-05: Teams Approval Card Action`
**Trigger:** Adaptive Card action submit in Teams.
**Action:** Invokes Reporting & Approval Agent with the decision payload.
**Failure handling:** If the Dataverse write fails, the card shows an error state and the human is asked to retry — a failed decision write must never silently appear as "recorded" to the committee member.

## `Flow-06: Weekly Remediation Sweep`
**Trigger:** Recurrence, weekly, Monday 08:00.
**Action:** Invokes Reporting & Approval Agent (`remediation_status_check`); posts escalation notifications for overdue actions.

## Design rule
Every flow that crosses the human-in-the-loop boundary (Flow-05 specifically) writes to Dataverse synchronously before returning success to the Teams UI — no "fire and forget" on a governance decision write.
