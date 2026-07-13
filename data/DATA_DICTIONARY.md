# Data Dictionary — Cross-Reference Index

**Audience:** anyone who needs "which table has field X" without reading all of `TABLE_CATALOG.md`.
**Note:** this is an index, not a duplicate spec. Full field definitions, types and business rules are authoritative in `TABLE_CATALOG.md`.

## 1. Key field index (fields that appear in multiple tables — resolves ambiguity)

| Field name | Appears in | Meaning is consistent? |
|---|---|---|
| `account_sk` | `dim_account`, `dim_account_revolving_ext`, `dim_account_termloan_ext`, `fact_account_snapshot_monthly`, `fact_model_score_event`, `fact_repayment_transaction` | Yes — always the surrogate key of `dim_account`, always joins back to exactly one current account row |
| `snapshot_date` | `fact_account_snapshot_monthly`, `fact_revolving_behaviour_monthly` (via join), `fact_termloan_behaviour_monthly` (via join), `fact_delinquency_state_monthly` | Yes — always month-end, always the B Score observation-point candidate date |
| `observation_date` | `fact_model_score_event`, `fact_performance_label` | Yes — but its *basis* differs by model family (application date for A, snapshot date for B, bucket-observation date for C) — see `TEMPORAL_MODEL.md` §2 |
| `effective_start_date` / `effective_end_date` | every SCD2 dimension (`dim_customer`, `dim_account`, `dim_account_revolving_ext`, `dim_governance_threshold`) | Yes — standard SCD2 semantics, null `effective_end_date` = current |
| `load_timestamp` | every table | Yes — always processing time, never used in business logic, only for pipeline debugging |
| `calculation_version` | `fact_bureau_aggregate_monthly`, `fact_monitoring_metric` | Yes — increments whenever the calculation logic (not just the input data) changes |
| `run_type` | `fact_model_score_event` | Values `live` / `backscore` — never assume `live` without checking |
| `maturity_status` | `fact_performance_label` | The single most important filter field in the schema — see ADR-006 |
| `rag_status` | `fact_monitoring_metric`, `fact_segment_monitoring_metric` | Computed from `dim_governance_threshold`, never set by an agent |
| `status` | `fact_investigation`, `fact_recommendation`, `dim_agent`, `dim_model`, `fact_remediation_action` | Distinct enumerations per table — see `TABLE_CATALOG.md` for the specific values per table; do not assume shared vocabulary across tables |

## 2. Sensitivity classification — field-level exceptions

Most sensitivity is table-level (see `TABLE_CATALOG.md` "Sensitivity" per table). Two field-level exceptions:

| Table | Field | Classification | Why different from table default |
|---|---|---|---|
| `fact_application` | `requested_amount` | Confidential (table default) | No exception — listed to clarify it is *not* Restricted despite being financial |
| `dim_customer` | `birth_year` | Restricted (table default), but year-only by design | Deliberately minimized from full DOB at the schema level, not masked downstream — see `dim_customer` spec |
| `fact_bureau_tradeline` | `outstanding_balance`, `credit_limit` | Restricted | External bureau financial data carries higher sensitivity than internal account data of the same shape |

## 3. Enumerated value catalogs

Full canonical lists live as governed rows, not hardcoded here — see:
- `dim_root_cause_taxonomy` for root cause codes
- `dim_governance_threshold` for bucket definitions, RAG bands, eligibility rules
- `dim_monitoring_segment` for segment dimension/value pairs

This is deliberate: enumerations that affect governance decisions are policy-as-code (ADR-007), not documentation-only.

## 4. Naming conventions

| Prefix | Meaning |
|---|---|
| `dim_` | Dimension — describes an entity, may be SCD2 |
| `fact_` | Fact — records an event or measurement, append-only |
| `_sk` suffix | Surrogate key, internally generated, meaningless outside the warehouse |
| `_id` suffix | Business/natural key, meaningful and stable across systems |
| `_ext` suffix | Product-family extension table (see `DOMAIN_MODEL.md` §2) |

## 5. Where to find what

| Question | Go to |
|---|---|
| "What does field X mean and what table is it in" | This file → `TABLE_CATALOG.md` |
| "How does MOB get calculated" | `TEMPORAL_MODEL.md` §7 |
| "What's the grain of table Y" | `TABLE_CATALOG.md`, table Y's "Grain" line |
| "What agents can read table Y" | `TABLE_CATALOG.md`, table Y's "Downstream agents" line, cross-checked against `architecture/SECURITY_ARCHITECTURE.md` §2 |
