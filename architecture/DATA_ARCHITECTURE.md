# Data Architecture

**Audience:** data architects, Copilot Studio builders needing to know where a given table lives.
**Depends on:** `data/DOMAIN_MODEL.md`, `data/TABLE_CATALOG.md`.

## 1. Layering (medallion, on Fabric + Dataverse)

| Layer | Platform | Contents | Update pattern |
|---|---|---|---|
| Bronze | Fabric Lakehouse | Raw synthetic source extracts: applications, snapshots, repayments, bureau files, as they would "arrive" from source systems | Append-only, immutable, one file per synthetic "batch date" |
| Silver | Fabric Lakehouse | Curated, conformed dimensional and fact tables: `dim_customer`, `dim_account` (+ extensions), `fact_account_snapshot_monthly` (+ extensions), `fact_bureau_tradeline`, `fact_repayment_transaction` | SCD2 merge for dimensions; append for facts, keyed by natural key + event date |
| Gold | Fabric Warehouse | Monitoring-ready deterministic outputs: `fact_model_score_event`, `fact_performance_label`, `fact_monitoring_metric`, `fact_segment_monitoring_metric`, `fact_data_quality_check` | Rebuilt/appended each monitoring run; every row versioned |
| Governance | Dataverse | Human-facing, agent-read/write: investigation, evidence, finding, recommendation, committee decision, agent/tool logs | Transactional CRUD via Copilot Studio actions |

Rationale for splitting Gold (Fabric Warehouse) from Governance (Dataverse) rather than one store: the monitoring mart is analytical, append-heavy, and SQL-native; the governance layer is transactional, human-approval-driven, and benefits from Dataverse's native row-level security and Copilot Studio low-code actions. Mixing them would force either analytical workloads into Dataverse (poor fit for large fact tables) or transactional approval workflows into a warehouse (poor fit for row-level security and Teams integration).

## 2. Product-extension pattern (applies across silver and gold)

Every domain where product mechanics materially differ uses: **one parent table at the common grain + one extension table per product family**, joined 1:1 on the parent's primary key. Applied to:

- `dim_account` → `dim_account_revolving_ext` (CC, SPC) / `dim_account_termloan_ext` (SPL)
- `fact_account_snapshot_monthly` → `fact_revolving_behaviour_monthly` (CC, SPC) / `fact_termloan_behaviour_monthly` (SPL)

A query that only needs common fields (balance, status, product code) never joins the extension. A query that needs utilization joins the revolving extension and gets a clean absence (no row), not a null column, for term loans. See `adr/ADR-009-mvp-product-model.md` for the rejected alternatives (single wide table; fully generic EAV model).

## 3. Model-family-specific structures

A Score, B Score and C Score share `fact_model_score_event` and `fact_performance_label` at the same grain (one row per model_id + account_id + score_date), but:
- A Score's `observation_date` = `application_date` (pre-booking); B/C Score's `observation_date` is post-booking.
- C Score alone carries a `delinquency_bucket_id` FK on both the score event and the label, because the eligible population and outcome definition are bucket-specific.
- These differences are expressed through **nullable-by-design, documented FKs** (e.g., `delinquency_bucket_id` is required for C Score rows and must be null for A/B Score rows — enforced by a Data Quality Agent check, not by separate physical tables), because the differences are attributes of the same event grain, not a different grain. This is the deliberate exception to "don't use nullable columns for product logic" — the distinction here is *model family*, which is a dimension of the same fact, not a materially different entity.

## 4. Retention and sensitivity summary

Full per-table detail is in `data/TABLE_CATALOG.md`. Summary bands:

| Sensitivity | Examples | Retention | Access |
|---|---|---|---|
| Restricted (PII) | `dim_customer` | 7 years post account closure (regulatory minimum) | Data Quality Agent (existence checks only, no field read), no other agent |
| Confidential (customer-linked, non-PII) | account/snapshot/score facts | 7 years | Monitoring, Investigation agents (aggregate/row-cited, not bulk export) |
| Internal (model/governance metadata) | model inventory, monitoring metrics, policy | Indefinite (governance record) | All agents |
| Internal (agent operational logs) | execution/tool logs | 2 years rolling | Assurance Agent, human audit only |

## 5. Why not a single generic schema

The previous schema iteration used one wide account table and one wide snapshot table for all products, and one generic "score" table for all model families. This repository replaces that with the parent/extension pattern above specifically because the brief's critical modeling distinctions (revolving vs term-loan mechanics, A/B/C observation points) are *business-meaningful differences in what a row means*, not cosmetic differences in which columns are populated. See `adr/ADR-009-mvp-product-model.md`.
