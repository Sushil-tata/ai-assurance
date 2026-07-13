# ERD (Part 3 of Final Output — Mermaid)

**Audience:** everyone; this is the visual companion to `TABLE_CATALOG.md`. Split into three diagrams because a single 45-table ERD is unreadable — each diagram is complete within its layer, with explicit cross-layer join keys called out below.

## 1. Layer A + B — Customer, account, behaviour (source-of-truth + time series)

```mermaid
erDiagram
    dim_customer ||--o{ fact_application : "customer_sk"
    dim_customer ||--o{ dim_account : "customer_sk"
    fact_application ||--o| dim_account : "application_id (booking)"
    dim_account ||--o| dim_account_revolving_ext : "account_sk (CC/SPC only)"
    dim_account ||--o| dim_account_termloan_ext : "account_sk (SPL only)"
    dim_account ||--o{ fact_account_snapshot_monthly : "account_sk"
    fact_account_snapshot_monthly ||--o| fact_revolving_behaviour_monthly : "account_snapshot_sk (revolving)"
    fact_account_snapshot_monthly ||--o| fact_termloan_behaviour_monthly : "account_snapshot_sk (term loan)"
    dim_account_termloan_ext ||--o{ fact_termloan_behaviour_monthly : "account_termloan_ext_sk"
    dim_account ||--o{ fact_repayment_transaction : "account_sk"
    fact_account_snapshot_monthly ||--o| fact_delinquency_state_monthly : "account_snapshot_sk"
    dim_customer ||--o{ fact_bureau_tradeline : "customer_sk"
    dim_customer ||--o{ fact_bureau_aggregate_monthly : "customer_sk"

    dim_customer {
        bigint customer_sk PK
        varchar customer_id NK
        varchar income_band
        boolean is_current
    }
    fact_application {
        varchar application_id PK
        bigint customer_sk FK
        varchar product_code
        date application_date "A Score obs point"
        date booking_date
    }
    dim_account {
        bigint account_sk PK
        varchar account_id NK
        bigint customer_sk FK
        varchar product_code
        varchar product_family
        date booking_date "MOB=0 basis"
    }
    dim_account_revolving_ext {
        bigint account_revolving_ext_sk PK
        bigint account_sk FK
        decimal current_credit_limit
    }
    dim_account_termloan_ext {
        bigint account_termloan_ext_sk PK
        bigint account_sk FK
        smallint original_tenor_months
    }
    fact_account_snapshot_monthly {
        bigint account_snapshot_sk PK
        bigint account_sk FK
        date snapshot_date "B Score obs point"
        smallint mob
    }
    fact_revolving_behaviour_monthly {
        bigint revolving_behaviour_sk PK
        bigint account_snapshot_sk FK
        decimal utilization_pct
        decimal payment_ratio
    }
    fact_termloan_behaviour_monthly {
        bigint termloan_behaviour_sk PK
        bigint account_snapshot_sk FK
        varchar maturity_stage
        smallint remaining_tenor_months
    }
    fact_delinquency_state_monthly {
        bigint delinquency_state_sk PK
        bigint account_snapshot_sk FK
        smallint bucket_current
        varchar transition_type
    }
    fact_repayment_transaction {
        varchar repayment_transaction_id PK
        bigint account_sk FK
        date transaction_date
    }
    fact_bureau_tradeline {
        bigint bureau_tradeline_sk PK
        bigint customer_sk FK
        date bureau_pull_date
        boolean is_missing_flag
    }
    fact_bureau_aggregate_monthly {
        bigint bureau_aggregate_sk PK
        bigint customer_sk FK
        decimal bureau_missingness_pct
    }
```

## 2. Layer C + D — Model, scoring, monitoring, governance

```mermaid
erDiagram
    dim_model ||--o{ dim_model_version : "model_id"
    dim_model_version ||--o{ fact_model_deployment : "model_version_id"
    dim_model_version ||--o{ dim_feature_definition : "feature_set_version_id"
    dim_model_version ||--o{ fact_model_score_event : "model_version_id"
    dim_account ||--o{ fact_model_score_event : "account_sk"
    fact_model_score_event ||--o{ fact_feature_lineage : "score_event_id"
    dim_feature_definition ||--o{ fact_feature_lineage : "feature_definition_sk"
    fact_model_score_event ||--o{ fact_performance_label : "score_event_id"
    dim_model_version ||--o{ fact_monitoring_metric : "model_version_id"
    dim_model_version ||--o{ fact_segment_monitoring_metric : "model_version_id"
    dim_monitoring_segment ||--o{ fact_segment_monitoring_metric : "segment_id"
    dim_policy_document ||--o{ dim_policy_clause : "policy_document_id"
    dim_policy_clause ||--o{ dim_governance_threshold : "policy_clause_id"
    dim_governance_threshold ||--o{ fact_monitoring_metric : "threshold_set_id"

    dim_model {
        varchar model_id PK
        varchar model_family "A / B / C"
        varchar product_code
        varchar lifecycle_stage
    }
    dim_model_version {
        varchar model_version_id PK
        varchar model_id FK
        varchar algorithm
        date validation_date
        date next_review_date
    }
    fact_model_deployment {
        varchar deployment_id PK
        varchar model_version_id FK
        varchar environment
        varchar deployment_status
    }
    dim_feature_definition {
        bigint feature_definition_sk PK
        varchar feature_set_version_id
        varchar feature_name
    }
    fact_model_score_event {
        varchar score_event_id PK
        varchar model_version_id FK
        bigint account_sk FK
        date observation_date
        smallint mob_at_observation
        smallint delinquency_bucket_id "C Score only"
        varchar run_type "live / backscore"
    }
    fact_feature_lineage {
        bigint feature_lineage_sk PK
        varchar score_event_id FK
        bigint feature_definition_sk FK
        boolean is_missing
    }
    fact_performance_label {
        bigint performance_label_sk PK
        varchar score_event_id FK
        smallint performance_window_months
        varchar maturity_status "mature/immature/censored"
    }
    fact_monitoring_metric {
        bigint monitoring_metric_sk PK
        varchar model_version_id FK
        varchar metric_code
        varchar rag_status
        boolean statistical_reliability_flag
    }
    fact_segment_monitoring_metric {
        bigint segment_monitoring_metric_sk PK
        varchar model_version_id FK
        varchar segment_id FK
        decimal population_share_pct
    }
    dim_monitoring_segment {
        varchar segment_id PK
        varchar segment_dimension
    }
    dim_policy_document {
        varchar policy_document_id PK
        varchar document_code
        date effective_date
    }
    dim_policy_clause {
        varchar policy_clause_id PK
        varchar policy_document_id FK
        varchar clause_number
    }
    dim_governance_threshold {
        varchar threshold_id PK
        varchar threshold_code
        varchar policy_clause_id FK
        varchar rule_type
    }
```

## 3. Layer E — Investigation, evidence, human decision, agent operations

```mermaid
erDiagram
    fact_monitoring_metric ||--o| fact_investigation : "trigger_metric_sk"
    fact_investigation ||--o{ fact_evidence_item : "investigation_id"
    fact_investigation ||--o{ fact_finding : "investigation_id"
    fact_finding ||--o{ fact_finding_evidence_link : "finding_id"
    fact_evidence_item ||--o{ fact_finding_evidence_link : "evidence_item_id"
    dim_root_cause_taxonomy ||--o{ fact_finding : "root_cause_code"
    fact_finding ||--o{ fact_finding_contributing_factor : "finding_id"
    dim_root_cause_taxonomy ||--o{ fact_finding_contributing_factor : "root_cause_code"
    fact_finding ||--o{ fact_recommendation : "finding_id"
    fact_investigation ||--o{ fact_human_approval : "investigation_id"
    fact_investigation ||--o| fact_committee_decision : "investigation_id"
    fact_recommendation ||--o| fact_committee_decision : "recommendation_id"
    fact_committee_decision ||--o{ fact_remediation_action : "decision_id"
    fact_recommendation ||--o{ fact_remediation_action : "recommendation_id"
    fact_committee_decision ||--o{ fact_agent_override : "decision_id"

    dim_agent ||--o{ fact_agent_execution_log : "agent_version_sk"
    dim_prompt_version ||--o{ dim_agent : "prompt_version_id"
    fact_agent_execution_log ||--o{ fact_tool_execution_log : "execution_id"
    fact_agent_execution_log ||--o{ fact_agent_evaluation : "execution_id"

    fact_investigation {
        varchar investigation_id PK
        varchar model_version_id FK
        varchar status
        date opened_date
        date closed_date
        varchar decision_id FK "required non-null before closed"
    }
    fact_evidence_item {
        varchar evidence_item_id PK
        varchar investigation_id FK
        varchar evidence_type
        boolean is_contradictory
        varchar confidence
    }
    fact_finding {
        varchar finding_id PK
        varchar investigation_id FK
        varchar root_cause_code FK
        varchar critic_review_status
    }
    dim_root_cause_taxonomy {
        varchar root_cause_code PK
        varchar root_cause_category
    }
    fact_recommendation {
        varchar recommendation_id PK
        varchar finding_id FK
        boolean requires_committee_approval
        varchar status
    }
    fact_committee_decision {
        varchar decision_id PK
        varchar investigation_id FK
        varchar decision
        varchar decided_by
        date decision_date
    }
    fact_remediation_action {
        varchar remediation_action_id PK
        varchar recommendation_id FK
        varchar status
    }
    dim_agent {
        bigint agent_version_sk PK
        varchar agent_id
        varchar agent_version
    }
    fact_agent_execution_log {
        varchar execution_id PK
        varchar correlation_id
        bigint agent_version_sk FK
        varchar status
    }
    fact_tool_execution_log {
        varchar tool_execution_id PK
        varchar execution_id FK
        varchar tool_id FK
    }
    fact_agent_evaluation {
        varchar evaluation_id PK
        bigint agent_version_sk FK
        varchar outcome
    }
```

## 4. Cross-layer join keys (how the three diagrams connect)

| From | To | Key |
|---|---|---|
| Diagram 1 → Diagram 2 | `dim_account.account_sk` → `fact_model_score_event.account_sk` | Every score event is for an account |
| Diagram 2 → Diagram 3 | `fact_monitoring_metric.monitoring_metric_sk` → `fact_investigation.trigger_metric_sk` | A breach opens an investigation |
| Diagram 2 → Diagram 3 | `dim_policy_clause.policy_clause_id` → `fact_evidence_item.policy_clause_id` | Policy citations become evidence |
| Diagram 1 → Diagram 3 | `fact_bureau_aggregate_monthly` / `fact_revolving_behaviour_monthly` rows → `fact_evidence_item.source_row_identifier` (documented, not FK-enforced across the Lakehouse/Dataverse boundary) | Raw behavioural rows cited as evidence |
