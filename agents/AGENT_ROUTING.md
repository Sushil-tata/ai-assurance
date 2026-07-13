# Agent Routing

**Audience:** Copilot Studio builders configuring the Orchestrator's connected-agent topics.

## 1. Intent-to-agent routing table

| User intent pattern | Routed to | Example utterance |
|---|---|---|
| Metric value / RAG status query | Monitoring Agent | "What's the current PSI for CC Behaviour Score?" |
| Data health / completeness query | Data Quality Agent | "Is the bureau feed healthy this month?" |
| Policy / threshold interpretation | Governance Agent | "What's the PSI amber threshold and where does it come from?" |
| "Why did X happen" / investigation status / follow-up | Investigation Agent | "Why is the PSI amber this month?" / "Show me investigation INV-2026-0031" |
| Review/quality-check request (rare, mostly internal) | Assurance/Critic Agent | "Has this finding been critic-reviewed?" |
| Report / approval / decision recording | Reporting & Approval Agent | "Show me the committee report for INV-2026-0031" / approval card action |
| Ambiguous / multi-intent | Orchestrator asks one clarifying question | "Tell me about the Behaviour Score" (could be metric, policy, or investigation) |

## 2. Event-driven (non-chat) routing

| Event | Source | Routed to |
|---|---|---|
| Scheduled monthly monitoring run | Power Automate timer | Monitoring Agent |
| `fact_investigation` row created | Dataverse trigger (Power Automate) | Investigation Agent |
| `fact_finding.critic_review_status = pending` | Dataverse trigger | Assurance/Critic Agent |
| `fact_finding.critic_review_status = approved` | Dataverse trigger | Reporting & Approval Agent |
| Teams adaptive card action | Teams/Power Automate | Reporting & Approval Agent |
| Weekly remediation sweep | Power Automate timer | Reporting & Approval Agent |

## 3. Hand-off payload contract

Every agent-to-agent hand-off (whether via Orchestrator or event trigger) carries at minimum: `correlation_id`, `investigation_id` (if applicable), `requesting_agent_id`, `request_type`. See `agents/AGENT_INPUT_OUTPUT_SCHEMAS.md` for full per-agent schemas.

## 4. Anti-patterns explicitly disallowed

- Investigation Agent calling Reporting & Approval Agent directly (must go through Assurance/Critic Agent first — no shortcut path exists in the Copilot Studio topic configuration).
- Any agent other than Reporting & Approval Agent posting a Teams approval card.
- Monitoring Agent calling Investigation Agent directly (decoupled via the Dataverse trigger, not a direct call, so that investigation creation is observable/auditable as a discrete event rather than buried inside a monitoring run's execution log).

## 5. Failure routing

If a specialist agent is unavailable (timeout, error), the Orchestrator does not silently retry indefinitely — one retry, then report the failure to the user/log, per each agent's "Error handling" section.
