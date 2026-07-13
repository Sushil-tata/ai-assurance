# Tool Catalog

**Audience:** Copilot Studio builders wiring actions; corresponds to `dim_tool` registry.
**MVP tool count:** 6 (within the 4–6 target in `SCOPE.md`).

| `tool_id` | Name | Type | Backing system | Used by | Read/Write | MVP? |
|---|---|---|---|---|---|---|
| `TOOL-GET-METRIC` | `get_monitoring_metric` | SQL action | Fabric Warehouse | Monitoring, Investigation | Read | Yes |
| `TOOL-GET-SEGMENT-METRIC` | `get_segment_metric` | SQL action | Fabric Warehouse | Monitoring, Investigation | Read | Yes |
| `TOOL-GET-DQ-CHECK` | `get_dq_check_result` | SQL action | Fabric Warehouse | Data Quality, Investigation | Read | Yes |
| `TOOL-SEARCH-POLICY` | `search_policy_clauses` | Azure AI Search action | Azure AI Search (policy index) | Governance, Investigation | Read | Yes |
| `TOOL-WRITE-EVIDENCE` | `write_evidence_item` | Dataverse action | Dataverse | Investigation, Data Quality | Write | Yes |
| `TOOL-INVESTIGATION-CRUD` | `create_investigation` / `update_investigation_status` | Dataverse action | Dataverse | Monitoring, Investigation | Read/Write | Yes |
| `TOOL-GET-THRESHOLD` | `get_governance_threshold` | Dataverse action | Dataverse | Governance | Read | Enterprise (MVP: folded into `TOOL-SEARCH-POLICY` result set for simplicity) |
| `TOOL-GET-FEATURE-LINEAGE` | `get_feature_lineage` | SQL action | Fabric Warehouse | Investigation | Read | Enterprise (MVP: CSI decomposition evidence pre-computed for the demo scenario rather than live-queried) |
| `TOOL-POST-TEAMS-CARD` | `post_teams_approval_card` | Power Automate flow | Teams | Reporting & Approval | Write (side-effect) | Yes |
| `TOOL-RECORD-DECISION` | `record_committee_decision` | Dataverse action | Dataverse | Reporting & Approval | Write | Yes |
| `TOOL-ASSEMBLE-REPORT` | `assemble_committee_report` | Power Automate flow + Power BI embed | Dataverse, Power BI | Reporting & Approval | Read | Yes |
| `TOOL-CHECK-APPLICABILITY` | `check_model_applicability` | SQL action | Fabric Warehouse | Monitoring | Read | Enterprise (MVP: applicability check is folded into the monthly monitoring SQL job's output, exposed via `TOOL-GET-METRIC`) |

**MVP six** (per `SCOPE.md`): `get_monitoring_metric`, `get_segment_metric`, `get_dq_check_result`, `search_policy_clauses`, `write_evidence_item`, `create_investigation`/`update_investigation_status` (counted as one CRUD tool) — plus the Reporting Agent's report/decision/card actions are Power Automate flows, catalogued separately in `POWER_AUTOMATE_FLOWS.md` rather than counted against the "agent tool" budget, since they are triggered post-approval, not part of the investigation reasoning loop.

## Tool design rules
1. Every tool is single-purpose — no tool both reads and writes across unrelated domains.
2. Every tool call is logged to `fact_tool_execution_log` automatically by the wrapper, not by agent-side logging (so a misbehaving agent can't skip logging).
3. Every write tool validates its caller's `agent_id` against the allow-list in `dim_agent.allowed_tools` before executing — enforced in the tool wrapper, not just documented.
