# Security Architecture

**Audience:** security review, Copilot Studio Solution Architect.
**Principle applied:** least privilege (design principle 1.8), enforced at three independent layers so that a failure in any one layer does not grant unintended access.

## 1. Three enforcement layers

| Layer | Mechanism | What it stops |
|---|---|---|
| Identity | Entra ID service principal per agent, scoped security role per Dataverse table / Fabric Warehouse schema | An agent authenticating with more privilege than its role needs |
| Connector | Copilot Studio action/connector allow-list per agent (only the actions listed in that agent's contract are wired into its topic) | An agent attempting to call a tool it was never given, even if it "knows" the tool exists from training |
| Application | Row/column-level security in Dataverse; view-level column masking in Fabric Warehouse for PII-adjacent fields | Bulk export of restricted fields even by an agent with legitimate table-level access |

## 2. Per-agent access matrix

| Agent | Fabric Warehouse (monitoring mart) | Dataverse (governance layer) | SharePoint / AI Search (policy) | `dim_customer` (PII) |
|---|---|---|---|---|
| Orchestrator | None (routes only) | Read (investigation status) | None | None |
| Monitoring Agent | Read `fact_monitoring_metric`, `fact_segment_monitoring_metric`, `fact_model_score_event`, `fact_performance_label` | Write `fact_investigation` (trigger creation only) | None | None |
| Data Quality Agent | Read `fact_data_quality_check`, pipeline completeness views | Write `fact_evidence_item` (DQ findings only) | None | Existence/row-count checks only, no field-level read |
| Governance Agent | Read `dim_governance_threshold`, `dim_model`, `dim_model_version` | Read/write `dim_policy_document`, `dim_policy_clause` | Read (search index) | None |
| Investigation Agent | Read all monitoring mart tables (read-only) | Read/write `fact_investigation`, `fact_evidence_item`, `fact_finding`, `fact_recommendation` | Read (search index) | None |
| Assurance / Critic Agent | Read (verification only) | Read all governance tables, write `fact_agent_evaluation` review outcome | Read (search index, verification) | None |
| Reporting & Approval Agent | Read approved/closed investigation aggregates only | Read/write `fact_committee_decision`, `fact_human_approval` | None | None |

No agent in this system has field-level read access to `dim_customer` beyond an existence/row-count check used solely by the Data Quality Agent to confirm referential completeness. No agent has write access to any silver-layer or bronze-layer table (`dim_account`, `fact_account_snapshot_monthly`, bureau tables) — those are pipeline-owned, not agent-owned.

## 3. Prompt-injection and tool-boundary containment

Because each agent's connector allow-list is enforced at the Copilot Studio topic configuration level (not only in the system prompt), a successful prompt injection against, say, the Reporting & Approval Agent cannot escalate to a Dataverse write on `fact_committee_decision` outside its own already-narrow write scope, and categorically cannot reach the Fabric Warehouse raw tables or any production system, because no connector for those exists in that agent's topic at all. This is the practical difference between "the prompt says don't do X" and "the platform has no action that can do X."

## 4. Sensitivity classification enforcement

Column-level sensitivity labels (Restricted / Confidential / Internal, per `data/TABLE_CATALOG.md`) are applied as Purview labels in the enterprise target and as documented, code-reviewed view definitions in the MVP (Purview automation is future-state per `architecture/SYSTEM_ARCHITECTURE.md`). Every view an agent reads through excludes Restricted columns by definition — the agent is never in a position to accidentally select a PII column because it does not exist in the view's schema.

## 5. Secrets and credentials

Fabric Warehouse and Dataverse connections use managed identities per environment (dev/demo/prod), never embedded credentials in flows or agent instructions. Azure AI Search API keys are stored in Azure Key Vault and referenced by the Function/connector, not pasted into Copilot Studio configuration.

## 6. What is explicitly not covered by MVP security build

Automated Purview lineage-based access reviews and Entra Conditional Access policies tuned specifically for this workload are enterprise-target items (see `MVP_VS_ENTERPRISE.md`). The MVP relies on manually configured Entra roles and documented (not automated) column masking — sufficient to demonstrate the architecture pattern, not sufficient for production go-live without the enterprise hardening pass.
