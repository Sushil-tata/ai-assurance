# API Contracts

**Audience:** Azure Function / API builders.

## `GET /metrics/{model_version_id}`
Query params: `metric_code`, `calculation_run_id` (optional, defaults to latest), `segment_id` (optional).
Returns: `fact_monitoring_metric` or `fact_segment_monitoring_metric` row(s) as JSON, matching the field list in `data/TABLE_CATALOG.md` Domain 16/17.
Backs: `TOOL-GET-METRIC`, `TOOL-GET-SEGMENT-METRIC`.
Auth: managed identity, Entra role `AIAssurance.Monitoring.Read`.

## `GET /dq-checks/{target_table}`
Query params: `run_id`, `rule_code` (optional).
Returns: `fact_data_quality_check` row(s).
Backs: `TOOL-GET-DQ-CHECK`.
Auth: `AIAssurance.DataQuality.Read`.

## `POST /evidence`
Body: `fact_evidence_item` field set (excluding surrogate key, generated server-side).
Validates: `investigation_id` exists and `status = open`; `collected_by_agent_id` is in the calling agent's own identity (no agent may write evidence attributed to another agent).
Backs: `TOOL-WRITE-EVIDENCE`.
Auth: `AIAssurance.Investigation.Write` (Investigation, Data Quality agents only).

## `POST /investigations`
Body: `trigger_type`, `trigger_metric_sk`, `trigger_description`, `severity`, `model_version_id`.
Returns: generated `investigation_id`.
Backs: `TOOL-INVESTIGATION-CRUD` (create).
Auth: `AIAssurance.Investigation.Create` (Monitoring, Data Quality agents only).

## `PATCH /investigations/{investigation_id}/status`
Body: `status` (enum, restricted transitions per `data/GOVERNANCE_SCHEMA.md` §3 state machine — server-side validated, not client-trusted).
Backs: `TOOL-INVESTIGATION-CRUD` (update).
Auth: `AIAssurance.Investigation.Write`; transition to `closed` additionally requires a valid `decision_id` already present or supplied in the same call, enforced server-side (ADR-005).

## `POST /policy-search`
Body: `free_text_query`, `top_k` (default 5).
Returns: ranked `dim_policy_clause` matches with relevance score, via Azure AI Search.
Backs: `TOOL-SEARCH-POLICY`.
Auth: `AIAssurance.Governance.Read`.

## `POST /committee-decisions`
Body: `investigation_id`, `recommendation_id`, `decision`, `decision_rationale`, `decided_by`.
Validates: `decided_by` resolves to a real Entra identity with Model Risk Committee role membership.
Backs: `TOOL-RECORD-DECISION`.
Auth: `AIAssurance.Reporting.Write` (Reporting & Approval Agent only).

## Common response envelope
```json
{ "status": "ok | error", "data": { }, "error": { "code": "string", "message": "string" } }
```

## Common error codes
| Code | Meaning |
|---|---|
| `INVALID_TRANSITION` | Requested status transition not allowed by the investigation state machine |
| `UNAUTHORIZED_AGENT` | Calling agent identity not on the allow-list for this action |
| `MISSING_DECISION` | Attempted to close an investigation without a `decision_id` |
| `NOT_FOUND` | Referenced ID does not exist |
| `STALE_INDEX` | Policy search index older than latest policy document effective date (warning, not hard failure) |
