# Error Handling — Cross-Cutting Rules

**Audience:** every tool/flow builder. These rules apply uniformly; per-tool exceptions are called out explicitly in `TOOL_CATALOG.md` / `SQL_ACTIONS.md` where they exist.

## 1. Principles

1. **Fail loud, never fail silent.** A tool that cannot complete its request returns an explicit error object; it never returns a plausible-looking empty/default result that an agent could mistake for a real answer.
2. **No inferred defaults on missing data.** If `fact_monitoring_metric` has no row for a requested period, the tool returns `NOT_FOUND`, not a zero or a copy of the prior period's value.
3. **Timeouts are short and explicit.** All SQL actions timeout at 10 seconds; Azure AI Search actions at 5 seconds. A timeout is logged to `fact_tool_execution_log` with `status = timeout`, distinct from `status = error`.
4. **Retries are bounded and logged.** Maximum one retry for transient failures (connection reset, throttling); no retry for validation errors (`INVALID_TRANSITION`, `UNAUTHORIZED_AGENT`) since retrying a request that will deterministically fail the same way wastes a call and obscures the log.
5. **Every error is attributable.** `fact_tool_execution_log.response_payload` always includes the structured error code/message from `API_CONTRACTS.md`, never just an HTTP status code.

## 2. Agent-level consequences of tool failure

| Failure scenario | Agent behavior |
|---|---|
| Monitoring Agent's metric query fails | Do not report a RAG status; report "unable to evaluate metric X this run" and create a DQ-flavoured investigation if the underlying cause is pipeline non-completion |
| Investigation Agent's evidence-gathering tool call fails | Downgrade the resulting finding's confidence; explicitly note the gap in `finding_statement` rather than omitting the attempted evidence silently |
| Governance Agent's search returns zero results | Report "not found in indexed policy library," never fall back to general LLM knowledge of "typical" policy language |
| Reporting Agent's Teams post fails | Retry once, then fall back to plain-text notification with direct link (per `agents/REPORTING_APPROVAL_AGENT.md`) |

## 3. What is explicitly NOT auto-recovered

- A `MISSING_DECISION` error on an attempted investigation closure is never auto-resolved by any agent — it is a structural block, not a transient failure.
- An `UNAUTHORIZED_AGENT` error is never retried with different credentials — it indicates a configuration bug (an agent's connector is wired to a tool outside its contract) and should page engineering, not be worked around at runtime.
