# Observability

**Audience:** builders debugging agent behaviour, audit reviewers reconstructing a decision trail.

## 1. What is logged, and where

| Signal | Store | Written by | Correlated by |
|---|---|---|---|
| Agent invocation (input, output, latency, model/prompt version) | `fact_agent_execution_log` (Dataverse) | Every agent, on every turn | `correlation_id` |
| Tool/action call (request, response, status, latency) | `fact_tool_execution_log` (Dataverse) | Every tool wrapper | `correlation_id`, `agent_execution_log_id` |
| Deterministic pipeline run (monitoring calculation batch) | `fact_monitoring_metric.calculation_run_id` + pipeline log table in Fabric | Azure Function / notebook | `calculation_run_id` |
| Distributed trace spans across agent → tool → Dataverse/Warehouse | Application Insights | Auto-instrumentation + custom spans tagged with `correlation_id` | `correlation_id` |
| Human decisions | `fact_committee_decision`, `fact_human_approval` | Reporting & Approval Agent, on human action | `investigation_id` |

`correlation_id` is generated once per end-user request (or once per scheduled monitoring run) and threaded through every downstream agent call, tool call, and log row — this is what lets a reviewer reconstruct "what happened" for a single investigation end to end, matching the evidence trace requirement in Part 6.

## 2. Minimum fields on every execution log row

`fact_agent_execution_log`: `execution_id` (PK), `correlation_id`, `agent_id` (FK `dim_agent`), `prompt_version_id` (FK `dim_prompt_version`), `invocation_time` (event time), `logged_time` (processing time), `input_payload` (JSON, redacted per sensitivity rules), `output_payload` (JSON), `status` (`success` / `error` / `escalated`), `error_detail`, `duration_ms`.

`fact_tool_execution_log`: `tool_execution_id` (PK), `correlation_id`, `execution_id` (FK, the agent call that triggered it), `tool_id` (FK `dim_tool` — see `tools/TOOL_CATALOG.md`), `request_payload`, `response_payload`, `status`, `duration_ms`, `invocation_time`, `logged_time`.

## 3. What Application Insights adds beyond the Dataverse logs

Dataverse logs are the **governance record** (queryable, retained per policy, human-reviewable in context). Application Insights is the **engineering diagnostic layer** — used to debug latency, connector failures, and throttling, and to alert engineering (not the Model Risk Committee) when the platform itself is unhealthy. The two are deliberately not merged: an audit reviewer should never need to query a raw trace store to reconstruct a governance decision, and an engineer debugging a timeout should never need Dataverse row-level security clearance to see a stack trace.

## 4. MVP versus future

MVP: Dataverse execution/tool logs (required — this is how the demo proves the evidence trace) + basic Application Insights auto-instrumentation (required for build debugging). **Future:** automated anomaly alerting on trace data (e.g., page an engineer when tool error rate exceeds X%), and a Power BI operational dashboard over the trace store, are enterprise increments not needed to prove the MVP narrative.
