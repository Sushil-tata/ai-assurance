# Agent Input/Output Schemas — Consolidated Reference

**Audience:** Copilot Studio builders wiring topic inputs/outputs.
**Note:** authoritative per-agent schemas are in each `agents/*.md` file; this is a consolidated index plus the shared envelope every agent call uses.

## 1. Shared envelope

Every agent invocation, regardless of specialist, is wrapped in:

```json
{
  "correlation_id": "string, required, generated once per end-user request or scheduled run",
  "requesting_agent_id": "string, required — AGT-ORCHESTRATOR or the calling specialist",
  "timestamp": "ISO 8601, required",
  "payload": { "...agent-specific input schema..." }
}
```

And every response:

```json
{
  "correlation_id": "string, echoed",
  "responding_agent_id": "string",
  "status": "success | error | escalated",
  "result": { "...agent-specific output schema..." },
  "error_detail": "string, present only if status=error"
}
```

## 2. Per-agent schema index

| Agent | Input schema location | Output schema location |
|---|---|---|
| Orchestrator | `agents/ORCHESTRATOR.md` §"Input schema" | same file, §"Output schema" |
| Monitoring | `agents/MONITORING_AGENT.md` | same |
| Data Quality | `agents/DATA_QUALITY_AGENT.md` | same |
| Governance | `agents/GOVERNANCE_AGENT.md` | same |
| Investigation | `agents/INVESTIGATION_AGENT.md` | same |
| Assurance/Critic | `agents/ASSURANCE_AGENT.md` | same |
| Reporting & Approval | `agents/REPORTING_APPROVAL_AGENT.md` | same |

Machine-readable JSON Schema files for the above (used for Copilot Studio action validation and automated testing) live in `schemas/agent_inputs/` and `schemas/agent_outputs/`, one file per agent, named `<agent_id>.schema.json`.

## 3. Versioning rule

Any change to an agent's input/output schema requires a `dim_prompt_version` bump (even if the prompt text itself didn't change) because the schema is part of the agent's contract with the rest of the system — a silent schema change without a version bump would break traceability between `fact_agent_execution_log.prompt_version_id` and the actual payload shape used at that time.
