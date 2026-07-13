# Table Catalog — Full Specification (Part 2)

**Audience:** data architects, ETL/pipeline builders, Copilot Studio tool authors.
**Rule:** every table below has exact grain, keys, fields, temporal logic, SCD type, source, retention, sensitivity, an example row, and named downstream agent consumers. Field types use ANSI SQL types as implemented in Fabric Warehouse / Dataverse.

Legend: **PK** primary key · **FK** foreign key · **NK** natural key · Req = required (NOT NULL).

---

# DOMAIN 1 — Customer and demographics

## `dim_customer`
**Business purpose:** single governed view of a customer, versioned over time (address/employment/income change), used to join to accounts and to check DQ existence — never read field-by-field by agents.
**Grain:** one row per customer per SCD2 version.
**Primary Key:** `customer_sk` (surrogate, bigint, identity)
**Natural Key:** `customer_id` (business key from source CRM)
**Foreign Keys:** none (root dimension)
**SCD Type:** 2 (tracks address, employment_status, income_band, marital_status changes)
**Source System:** Synthetic CRM generator (`data/SYNTHETIC_DATA_SPEC.md` §2.1); enterprise: core banking CRM
**Retention:** 7 years post last account closure (regulatory minimum)
**Sensitivity:** **Restricted (PII)**

| Field | Type | Req | Notes |
|---|---|---|---|
| `customer_sk` | bigint | Y | PK |
| `customer_id` | varchar(20) | Y | NK, stable across SCD2 versions |
| `birth_year` | smallint | Y | Year only, not full DOB — minimization by design |
| `gender` | varchar(10) | N | Optional, synthetic distribution only |
| `employment_status` | varchar(30) | Y | e.g. `salaried`, `self_employed`, `unemployed` |
| `income_band` | varchar(20) | Y | Banded, never exact income |
| `province` | varchar(50) | Y | Geography for segment monitoring |
| `marital_status` | varchar(20) | N | |
| `customer_since_date` | date | Y | Event time: first relationship date |
| `effective_start_date` | date | Y | SCD2 |
| `effective_end_date` | date | N | SCD2, null = current |
| `is_current` | boolean | Y | SCD2 convenience flag |
| `source_system` | varchar(30) | Y | |
| `load_timestamp` | timestamp | Y | Processing time |

**Example row:** `{customer_sk: 100482, customer_id: "CUST-0048291", birth_year: 1989, employment_status: "salaried", income_band: "THB_30-50K", province: "Bangkok", customer_since_date: "2022-01-14", effective_start_date: "2024-03-01", effective_end_date: null, is_current: true}`
**Downstream agents:** Data Quality Agent (existence/row-count checks only). No other agent reads this table.

---

# DOMAIN 2 — Application and acquisition

## `fact_application`
**Business purpose:** one row per credit application; the observation record for A Score; source of acquisition channel/campaign attributes used in monitoring segment cuts (e.g., thin-file mix after a campaign — see Scenario 001).
**Grain:** one row per application.
**Primary Key:** `application_id` (varchar, business key generated at application)
**Foreign Keys:** `customer_sk` → `dim_customer`; `product_code` → `dim_product` (reference table, values: `CC`, `SPC`, `SPL`)
**Natural Key:** `application_id`
**SCD Type:** N/A (immutable fact once decisioned; corrections go through a new row + `superseded_by_application_id`, never an update)
**Source System:** Synthetic origination generator; enterprise: LOS (loan origination system)
**Retention:** 7 years
**Sensitivity:** Confidential

| Field | Type | Req | Notes |
|---|---|---|---|
| `application_id` | varchar(20) | Y | PK/NK |
| `customer_sk` | bigint | Y | FK `dim_customer` (as-of application date version) |
| `product_code` | varchar(10) | Y | `CC` / `SPC` / `SPL` |
| `application_date` | date | Y | **Event time — A Score observation point** |
| `application_channel` | varchar(30) | Y | e.g. `branch`, `digital`, `campaign_partner` |
| `campaign_id` | varchar(20) | N | FK-like reference, null if not campaign-sourced |
| `requested_amount` | decimal(14,2) | Y | |
| `decision` | varchar(20) | Y | `approved` / `declined` / `referred` |
| `decision_date` | date | Y | |
| `a_score_value` | decimal(9,6) | N | Denormalized copy of the A Score result for convenience; canonical value lives in `fact_model_score_event` |
| `booking_date` | date | N | Null if declined; set when account is opened — **this is the FK bridge to `dim_account`** |
| `load_timestamp` | timestamp | Y | Processing time |

**Example row:** `{application_id: "APP-2026-0441207", customer_sk: 100482, product_code: "CC", application_date: "2025-11-03", application_channel: "campaign_partner", campaign_id: "CMP-2025-THINFILE-04", requested_amount: 50000.00, decision: "approved", decision_date: "2025-11-04", booking_date: "2025-11-10"}`
**Downstream agents:** Monitoring Agent (A Score population construction), Investigation Agent (campaign-mix hypothesis testing, per Scenario 001 cause 1).

---

# DOMAIN 3 — Product and account master

## `dim_account` (common parent)
**Business purpose:** the single account record shared by all three products at a product-agnostic grain — status, balance, product code, key dates. This is the join point every cross-product query starts from.
**Grain:** one row per account per SCD2 version.
**Primary Key:** `account_sk` (surrogate)
**Natural Key:** `account_id`
**Foreign Keys:** `customer_sk` → `dim_customer`; `application_id` → `fact_application`
**SCD Type:** 2 (tracks `account_status`, `credit_limit` for revolving, closure)
**Source System:** Synthetic account generator; enterprise: core banking / loan servicing
**Retention:** 7 years post closure
**Sensitivity:** Confidential

| Field | Type | Req | Notes |
|---|---|---|---|
| `account_sk` | bigint | Y | PK |
| `account_id` | varchar(20) | Y | NK |
| `customer_sk` | bigint | Y | FK |
| `application_id` | varchar(20) | Y | FK |
| `product_code` | varchar(10) | Y | `CC` / `SPC` / `SPL` — determines which extension table has a matching row |
| `product_family` | varchar(15) | Y | `revolving` / `term_loan` — derived, denormalized for query convenience |
| `booking_date` | date | Y | **Event time — account origin; MOB = 0 basis** |
| `account_status` | varchar(20) | Y | `active` / `closed_paid` / `closed_charged_off` / `closed_other` |
| `closure_date` | date | N | Null while active |
| `closure_reason` | varchar(30) | N | |
| `effective_start_date` | date | Y | SCD2 |
| `effective_end_date` | date | N | SCD2 |
| `is_current` | boolean | Y | |
| `load_timestamp` | timestamp | Y | Processing time |

**Example row:** `{account_sk: 55231, account_id: "ACC-CC-0091823", customer_sk: 100482, application_id: "APP-2026-0441207", product_code: "CC", product_family: "revolving", booking_date: "2025-11-10", account_status: "active", effective_start_date: "2026-06-01", is_current: true}`
**Downstream agents:** Monitoring Agent, Investigation Agent (population construction, MOB filtering).

## `dim_account_revolving_ext`
**Business purpose:** static/slow-changing revolving-only attributes (credit limit history root, card product tier). Populated only for CC/SPC.
**Grain:** one row per revolving account per SCD2 version (1:1 with `dim_account` current version).
**Primary Key:** `account_revolving_ext_sk`
**Foreign Keys:** `account_sk` → `dim_account` (1:1, unique)
**Natural Key:** `account_id`
**SCD Type:** 2 (tracks `credit_limit`, `card_tier`)
**Source System:** Synthetic account generator
**Retention:** 7 years post closure
**Sensitivity:** Confidential

| Field | Type | Req | Notes |
|---|---|---|---|
| `account_revolving_ext_sk` | bigint | Y | PK |
| `account_sk` | bigint | Y | FK, unique |
| `card_tier` | varchar(20) | Y | e.g. `classic`, `platinum` (CC only, null for SPC) |
| `original_credit_limit` | decimal(14,2) | Y | |
| `current_credit_limit` | decimal(14,2) | Y | |
| `pricing_apr` | decimal(6,3) | Y | |
| `effective_start_date` | date | Y | |
| `effective_end_date` | date | N | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{account_revolving_ext_sk: 40112, account_sk: 55231, card_tier: "classic", original_credit_limit: 50000.00, current_credit_limit: 65000.00, pricing_apr: 24.0}`
**Downstream agents:** Monitoring Agent (utilization feature construction context), Investigation Agent.

## `dim_account_termloan_ext`
**Business purpose:** static term-loan attributes (original schedule) for SPL. No row exists for CC/SPC accounts.
**Grain:** one row per term-loan account (immutable — origination terms don't change via SCD; a restructure creates a *new* schedule version, tracked in `fact_termloan_behaviour_monthly`, not here).
**Primary Key:** `account_termloan_ext_sk`
**Foreign Keys:** `account_sk` → `dim_account` (1:1, unique)
**Natural Key:** `account_id`
**SCD Type:** 0 (immutable origination record)
**Source System:** Synthetic account generator
**Retention:** 7 years post closure
**Sensitivity:** Confidential

| Field | Type | Req | Notes |
|---|---|---|---|
| `account_termloan_ext_sk` | bigint | Y | PK |
| `account_sk` | bigint | Y | FK, unique |
| `original_principal` | decimal(14,2) | Y | |
| `original_tenor_months` | smallint | Y | |
| `instalment_amount` | decimal(14,2) | Y | Scheduled fixed instalment at origination |
| `interest_rate` | decimal(6,3) | Y | |
| `first_instalment_date` | date | Y | |
| `scheduled_maturity_date` | date | Y | `first_instalment_date + original_tenor_months` |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{account_termloan_ext_sk: 8821, account_sk: 55890, original_principal: 120000.00, original_tenor_months: 36, instalment_amount: 3934.20, interest_rate: 18.0, first_instalment_date: "2024-02-05", scheduled_maturity_date: "2027-01-05"}`
**Downstream agents:** Monitoring Agent (maturity-stage segment cuts), Investigation Agent.

---

# DOMAIN 4 — Monthly account snapshots

## `fact_account_snapshot_monthly` (common parent)
**Business purpose:** the monthly heartbeat of every account — balance, DPD, status — at a product-agnostic grain. This is the **B Score observation point** and the backbone every monitoring metric ultimately rolls up from.
**Grain:** one row per account per snapshot month (`snapshot_date` = last calendar day of month).
**Primary Key:** `account_snapshot_sk`
**Foreign Keys:** `account_sk` → `dim_account`
**Natural Key:** (`account_id`, `snapshot_date`)
**SCD Type:** N/A (fact; append-only per month, corrections via restatement — see `TEMPORAL_MODEL.md` §6)
**Source System:** Synthetic monthly snapshot generator; enterprise: servicing system month-end extract
**Retention:** 7 years
**Sensitivity:** Confidential

| Field | Type | Req | Notes |
|---|---|---|---|
| `account_snapshot_sk` | bigint | Y | PK |
| `account_sk` | bigint | Y | FK |
| `account_id` | varchar(20) | Y | NK part |
| `snapshot_date` | date | Y | **Event time — B Score observation point candidate** |
| `mob` | smallint | Y | Months on book as of `snapshot_date`, computed `DATEDIFF(month, booking_date, snapshot_date)` |
| `account_status` | varchar(20) | Y | As of snapshot |
| `current_balance` | decimal(14,2) | Y | |
| `dpd` | smallint | Y | Days past due as of snapshot |
| `delinquency_bucket_id` | smallint | N | FK `dim_governance_threshold` bucket definition; null if current |
| `is_closed_this_month` | boolean | Y | |
| `restatement_of_snapshot_sk` | bigint | N | Self-FK; non-null if this row corrects a prior snapshot |
| `load_timestamp` | timestamp | Y | Processing time |

**Example row:** `{account_snapshot_sk: 991204, account_sk: 55231, account_id: "ACC-CC-0091823", snapshot_date: "2026-06-30", mob: 7, account_status: "active", current_balance: 32100.00, dpd: 0, delinquency_bucket_id: null}`
**Downstream agents:** Monitoring Agent, Investigation Agent, Data Quality Agent (completeness checks).

---

# DOMAIN 5 — Revolving credit behaviour

## `fact_revolving_behaviour_monthly`
**Business purpose:** revolving-specific monthly mechanics — utilization, payment ratio, drawdown — required for CC/SPC B Score features and monitoring. No row for SPL.
**Grain:** one row per revolving account per snapshot month (1:1 with `fact_account_snapshot_monthly` where `product_family = 'revolving'`).
**Primary Key:** `revolving_behaviour_sk`
**Foreign Keys:** `account_snapshot_sk` → `fact_account_snapshot_monthly` (1:1, unique)
**Natural Key:** (`account_id`, `snapshot_date`)
**SCD Type:** N/A (fact)
**Source System:** Synthetic generator
**Retention:** 7 years
**Sensitivity:** Confidential

| Field | Type | Req | Notes |
|---|---|---|---|
| `revolving_behaviour_sk` | bigint | Y | PK |
| `account_snapshot_sk` | bigint | Y | FK, unique |
| `credit_limit` | decimal(14,2) | Y | As of snapshot (may differ from origination via limit management) |
| `revolving_balance` | decimal(14,2) | Y | Balance not paid in full, carried forward |
| `utilization_pct` | decimal(6,3) | Y | `current_balance / credit_limit` |
| `payment_amount` | decimal(14,2) | Y | Amount paid this cycle |
| `minimum_due` | decimal(14,2) | Y | |
| `payment_ratio` | decimal(6,3) | Y | `payment_amount / minimum_due` |
| `is_min_due_met` | boolean | Y | |
| `drawdown_amount` | decimal(14,2) | Y | New spend/cash advance this cycle |
| `cash_advance_flag` | boolean | Y | Whether any cash-advance drawdown occurred |
| `over_limit_flag` | boolean | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{revolving_behaviour_sk: 771290, account_snapshot_sk: 991204, credit_limit: 65000.00, revolving_balance: 32100.00, utilization_pct: 0.494, payment_amount: 4000.00, minimum_due: 1605.00, payment_ratio: 2.49, is_min_due_met: true, drawdown_amount: 6200.00, cash_advance_flag: false, over_limit_flag: false}`
**Downstream agents:** Monitoring Agent (feature drift on utilization/payment ratio), Investigation Agent.

---

# DOMAIN 6 — Term-loan behaviour

## `fact_termloan_behaviour_monthly`
**Business purpose:** term-loan-specific monthly mechanics — scheduled vs. actual instalment, amortization, remaining tenor, maturity stage, prepayment. No row for CC/SPC.
**Grain:** one row per term-loan account per snapshot month (1:1 with `fact_account_snapshot_monthly` where `product_family = 'term_loan'`).
**Primary Key:** `termloan_behaviour_sk`
**Foreign Keys:** `account_snapshot_sk` → `fact_account_snapshot_monthly` (1:1, unique); `account_termloan_ext_sk` → `dim_account_termloan_ext`
**Natural Key:** (`account_id`, `snapshot_date`)
**SCD Type:** N/A (fact)
**Source System:** Synthetic generator
**Retention:** 7 years
**Sensitivity:** Confidential

| Field | Type | Req | Notes |
|---|---|---|---|
| `termloan_behaviour_sk` | bigint | Y | PK |
| `account_snapshot_sk` | bigint | Y | FK, unique |
| `account_termloan_ext_sk` | bigint | Y | FK |
| `scheduled_instalment` | decimal(14,2) | Y | Per original schedule (or restructured schedule if `is_restructured`) |
| `actual_payment` | decimal(14,2) | Y | |
| `principal_component` | decimal(14,2) | Y | Amortization split |
| `interest_component` | decimal(14,2) | Y | |
| `outstanding_principal` | decimal(14,2) | Y | |
| `remaining_tenor_months` | smallint | Y | |
| `maturity_stage` | varchar(20) | Y | `early` (\<25% elapsed) / `mid` / `late` (\>75% elapsed) / `matured` |
| `is_prepayment` | boolean | Y | Actual payment exceeds scheduled instalment beyond tolerance |
| `prepayment_amount` | decimal(14,2) | N | |
| `is_restructured` | boolean | Y | TDR flag |
| `restructure_effective_date` | date | N | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{termloan_behaviour_sk: 66210, account_snapshot_sk: 991510, scheduled_instalment: 3934.20, actual_payment: 3934.20, principal_component: 2801.10, interest_component: 1133.10, outstanding_principal: 41200.00, remaining_tenor_months: 6, maturity_stage: "late", is_prepayment: false, is_restructured: false}`
**Downstream agents:** Monitoring Agent (maturity-stage segment monitoring), Investigation Agent.

---

# DOMAIN 7 — Repayment history

## `fact_repayment_transaction`
**Business purpose:** transaction-grain repayment ledger underlying the monthly aggregates above; needed for roll-rate/cure-rate calculation precision and for investigation drill-down to an individual payment event.
**Grain:** one row per repayment transaction.
**Primary Key:** `repayment_transaction_id`
**Foreign Keys:** `account_sk` → `dim_account`
**Natural Key:** `repayment_transaction_id` (source-system generated)
**SCD Type:** N/A (immutable ledger; reversals are new offsetting rows, never updates)
**Source System:** Synthetic payments generator; enterprise: payment gateway / core banking ledger
**Retention:** 7 years
**Sensitivity:** Confidential

| Field | Type | Req | Notes |
|---|---|---|---|
| `repayment_transaction_id` | varchar(30) | Y | PK/NK |
| `account_sk` | bigint | Y | FK |
| `transaction_date` | date | Y | **Event time** |
| `posting_date` | date | Y | Processing time (may lag `transaction_date`) |
| `amount` | decimal(14,2) | Y | |
| `payment_channel` | varchar(20) | Y | e.g. `auto_debit`, `counter`, `mobile` |
| `is_reversal` | boolean | Y | |
| `reversed_transaction_id` | varchar(30) | N | Self-FK if `is_reversal` |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{repayment_transaction_id: "PAY-20260628-889213", account_sk: 55231, transaction_date: "2026-06-28", posting_date: "2026-06-28", amount: 4000.00, payment_channel: "auto_debit", is_reversal: false}`
**Downstream agents:** Investigation Agent (drill-down only, via aggregated tool output, never bulk row export).

---

# DOMAIN 8 — Delinquency and collections state

## `fact_delinquency_state_monthly`
**Business purpose:** the C Score observation record — bucket state, transition, and outcome per account per month. Distinct from `fact_account_snapshot_monthly.delinquency_bucket_id` (a denormalized pointer) because this table carries the full collections-state detail (contact history flag, promise-to-pay, restructuring flag) that C Score and roll-rate/cure-rate calculations need.
**Grain:** one row per account per snapshot month where `dpd > 0` (or was `> 0` in the prior month, to capture cure in the same month it happens).
**Primary Key:** `delinquency_state_sk`
**Foreign Keys:** `account_snapshot_sk` → `fact_account_snapshot_monthly`
**Natural Key:** (`account_id`, `snapshot_date`)
**SCD Type:** N/A (fact)
**Source System:** Synthetic generator; enterprise: collections system
**Retention:** 7 years
**Sensitivity:** Confidential

| Field | Type | Req | Notes |
|---|---|---|---|
| `delinquency_state_sk` | bigint | Y | PK |
| `account_snapshot_sk` | bigint | Y | FK |
| `bucket_current` | smallint | Y | 0 = 1–29 DPD, 1 = 30–59 DPD, etc.; boundaries in `dim_governance_threshold` |
| `bucket_prior_month` | smallint | N | Null for first delinquency month |
| `transition_type` | varchar(20) | Y | `new_delinquent` / `cure` / `remain` / `roll_forward` / `npl` / `charge_off` / `restructure` |
| `days_past_due` | smallint | Y | |
| `promise_to_pay_flag` | boolean | Y | |
| `contact_made_flag` | boolean | Y | |
| `collections_treatment_code` | varchar(20) | N | |
| `is_npl` | boolean | Y | Non-performing threshold reached |
| `is_charged_off` | boolean | Y | |
| `charge_off_date` | date | N | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{delinquency_state_sk: 44120, account_snapshot_sk: 991890, bucket_current: 1, bucket_prior_month: 0, transition_type: "roll_forward", days_past_due: 41, promise_to_pay_flag: false, contact_made_flag: true, is_npl: false, is_charged_off: false}`
**Downstream agents:** Monitoring Agent (roll-rate/cure-rate calc inputs), Investigation Agent.

---

# DOMAIN 9 — Bureau tradelines

## `fact_bureau_tradeline`
**Business purpose:** external bureau-reported tradeline detail per pull, feeding bureau aggregate features used by A/B Score. Central to Scenario 001 cause 2 (bureau feature missingness).
**Grain:** one row per customer per bureau pull date per reported tradeline.
**Primary Key:** `bureau_tradeline_sk`
**Foreign Keys:** `customer_sk` → `dim_customer`
**Natural Key:** (`customer_id`, `bureau_pull_date`, `tradeline_source_id`)
**SCD Type:** N/A (fact, one snapshot per pull)
**Source System:** Synthetic bureau generator; enterprise: NCB/bureau file feed
**Retention:** 5 years (bureau-specific regulatory retention, shorter than account retention)
**Sensitivity:** **Restricted** (external bureau data, high sensitivity)

| Field | Type | Req | Notes |
|---|---|---|---|
| `bureau_tradeline_sk` | bigint | Y | PK |
| `customer_sk` | bigint | Y | FK |
| `bureau_pull_date` | date | Y | **Event time** |
| `tradeline_source_id` | varchar(30) | Y | Bureau-assigned tradeline identifier |
| `reporting_institution` | varchar(50) | N | |
| `tradeline_type` | varchar(20) | Y | `credit_card` / `personal_loan` / `mortgage` / etc. |
| `outstanding_balance` | decimal(14,2) | N | Nullable — a missing value here (vs. zero) is the exact signal investigated in Scenario 001 |
| `credit_limit` | decimal(14,2) | N | |
| `dpd_at_pull` | smallint | N | |
| `account_age_months` | smallint | N | |
| `is_missing_flag` | boolean | Y | Explicit flag distinguishing "bureau returned no data for this field" from "zero" — set by the ingestion pipeline, never inferred downstream |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{bureau_tradeline_sk: 220981, customer_sk: 100482, bureau_pull_date: "2026-06-05", tradeline_source_id: "BRU-TL-88213", tradeline_type: "credit_card", outstanding_balance: null, is_missing_flag: true}`
**Downstream agents:** Investigation Agent (missingness root-cause tracing), Data Quality Agent.

---

# DOMAIN 10 — Bureau aggregates

## `fact_bureau_aggregate_monthly`
**Business purpose:** customer-level rollup of bureau tradelines into the aggregate features actually consumed by scoring (total exposure, number of tradelines, worst delinquency across bureau). Separated from raw tradelines because this is a derived, versioned calculation (has its own `calculation_version`), not a source fact.
**Grain:** one row per customer per bureau aggregation month.
**Primary Key:** `bureau_aggregate_sk`
**Foreign Keys:** `customer_sk` → `dim_customer`
**Natural Key:** (`customer_id`, `aggregation_month`)
**SCD Type:** N/A (fact, append-only, restated via `calculation_version` bump if the aggregation logic changes)
**Source System:** Derived from `fact_bureau_tradeline` by a versioned pipeline
**Retention:** 5 years
**Sensitivity:** Restricted

| Field | Type | Req | Notes |
|---|---|---|---|
| `bureau_aggregate_sk` | bigint | Y | PK |
| `customer_sk` | bigint | Y | FK |
| `aggregation_month` | date | Y | **Event time (month-end)** |
| `total_bureau_exposure` | decimal(16,2) | N | Sum across tradelines; null if underlying data insufficient |
| `active_tradeline_count` | smallint | N | |
| `worst_dpd_across_bureau` | smallint | N | |
| `bureau_missingness_pct` | decimal(6,3) | Y | Share of expected tradeline fields that were `is_missing_flag = true` — **the direct DQ signal for Scenario 001** |
| `calculation_version` | varchar(10) | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{bureau_aggregate_sk: 71029, customer_sk: 100482, aggregation_month: "2026-06-01", total_bureau_exposure: null, active_tradeline_count: 2, worst_dpd_across_bureau: 15, bureau_missingness_pct: 0.42, calculation_version: "v3"}`
**Downstream agents:** Monitoring Agent (feature drift/missingness metric), Investigation Agent, Data Quality Agent.

---

# DOMAIN 11 — Model inventory and lifecycle

## `dim_model`
**Business purpose:** the model registry — one row per distinct model (not per version). Full lifecycle-stage detail in `data/MODEL_INVENTORY_SCHEMA.md` (Part 4); this row is the anchor every version and score event hangs off.
**Grain:** one row per model.
**Primary Key:** `model_id` (business key, e.g. `CC-BSCORE`)
**Foreign Keys:** none (root)
**Natural Key:** `model_id`
**SCD Type:** 2 (tracks `lifecycle_stage`, `model_owner`, `status`)
**Source System:** Model Risk model registry (manually maintained in MVP, Dataverse table)
**Retention:** Indefinite (permanent governance record)
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `model_id` | varchar(20) | Y | PK/NK, e.g. `CC-BSCORE` |
| `model_name` | varchar(100) | Y | |
| `product_code` | varchar(10) | Y | `CC` / `SPC` / `SPL` |
| `model_family` | varchar(5) | Y | `A` / `B` / `C` |
| `model_subtype` | varchar(30) | N | e.g. `bucket_0`, `bucket_1` for C Score |
| `lifecycle_stage` | varchar(30) | Y | See `MODEL_INVENTORY_SCHEMA.md` §1 |
| `materiality_tier` | varchar(10) | Y | `tier_1` / `tier_2` / `tier_3` |
| `model_owner` | varchar(100) | Y | |
| `status` | varchar(20) | Y | `active` / `retired` / `challenger` |
| `effective_start_date` | date | Y | |
| `effective_end_date` | date | N | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{model_id: "CC-BSCORE", model_name: "Credit Card Behaviour Score", product_code: "CC", model_family: "B", lifecycle_stage: "production_monitoring", materiality_tier: "tier_1", model_owner: "Retail Risk Analytics", status: "active"}`
**Downstream agents:** Monitoring Agent, Governance Agent, Investigation Agent, Reporting & Approval Agent.

---

# DOMAIN 12 — Model versions and deployments

## `dim_model_version`
**Business purpose:** every version of every model, each independently validated/approved — the level at which a score event is actually produced.
**Grain:** one row per model version.
**Primary Key:** `model_version_id`
**Foreign Keys:** `model_id` → `dim_model`; `feature_set_version_id` → `dim_feature_definition` (version group); `predecessor_model_version_id` self-FK
**Natural Key:** (`model_id`, `version_label`)
**SCD Type:** 0 (immutable once validated; a change creates a new version row)
**Source System:** Model Risk validation record (Dataverse)
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `model_version_id` | varchar(30) | Y | PK, e.g. `CC-BSCORE-v2.1` |
| `model_id` | varchar(20) | Y | FK |
| `version_label` | varchar(10) | Y | `v2.1` |
| `algorithm` | varchar(30) | Y | e.g. `gradient_boosted_trees` |
| `feature_set_version_id` | varchar(20) | Y | FK `dim_feature_definition` |
| `validator` | varchar(100) | Y | |
| `approver` | varchar(100) | Y | |
| `validation_date` | date | Y | |
| `next_review_date` | date | Y | |
| `predecessor_model_version_id` | varchar(30) | N | Self-FK |
| `successor_model_version_id` | varchar(30) | N | Self-FK, populated on retirement |
| `challenger_model_version_id` | varchar(30) | N | Self-FK, points to a version currently shadow-scoring |
| `policy_mapping_id` | varchar(20) | Y | FK `dim_policy_clause` (governing monitoring policy) |
| `threshold_set_id` | varchar(20) | Y | FK `dim_governance_threshold` (version group) |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{model_version_id: "CC-BSCORE-v2.1", model_id: "CC-BSCORE", version_label: "v2.1", algorithm: "gradient_boosted_trees", feature_set_version_id: "FS-CC-B-v14", validator: "Model Validation Team", approver: "Model Risk Committee", validation_date: "2025-09-15", next_review_date: "2026-09-15"}`
**Downstream agents:** Monitoring Agent, Investigation Agent (version-mismatch detection, Scenario 004), Governance Agent.

## `fact_model_deployment`
**Business purpose:** records where and when a model version is actually running — the operational counterpart to validation. Needed to answer "which endpoint produced this score" and to catch a deployed-vs-validated mismatch.
**Grain:** one row per model version per deployment environment per deployment event.
**Primary Key:** `deployment_id`
**Foreign Keys:** `model_version_id` → `dim_model_version`
**Natural Key:** (`model_version_id`, `environment`, `deployed_at`)
**SCD Type:** N/A (append-only deployment event log)
**Source System:** Deployment pipeline (manually logged in MVP)
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `deployment_id` | varchar(30) | Y | PK |
| `model_version_id` | varchar(30) | Y | FK |
| `environment` | varchar(20) | Y | `dev` / `demo` / `production` |
| `production_endpoint` | varchar(200) | N | |
| `deployed_at` | timestamp | Y | Event time |
| `deployed_by` | varchar(100) | Y | |
| `deployment_status` | varchar(20) | Y | `active` / `superseded` / `rolled_back` |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{deployment_id: "DEPLOY-CCB-0091", model_version_id: "CC-BSCORE-v2.1", environment: "production", production_endpoint: "scoring-api/cc-bscore/v2_1", deployed_at: "2025-10-01T02:00:00Z", deployment_status: "active"}`
**Downstream agents:** Investigation Agent (Scenario 004 version-mismatch checks), Governance Agent.

---

# DOMAIN 13 — Feature definitions and feature lineage

## `dim_feature_definition`
**Business purpose:** the governed catalog of every feature a model can consume — definition, source table, calculation logic reference — grouped into versioned feature sets.
**Grain:** one row per feature per feature-set version.
**Primary Key:** `feature_definition_sk`
**Foreign Keys:** none (references source tables by name, documented not FK-enforced, since source tables span both Lakehouse and Warehouse)
**Natural Key:** (`feature_set_version_id`, `feature_name`)
**SCD Type:** 0 (immutable per version; changing a definition creates a new feature-set version)
**Source System:** Feature engineering pipeline documentation (Dataverse)
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `feature_definition_sk` | bigint | Y | PK |
| `feature_set_version_id` | varchar(20) | Y | NK part, e.g. `FS-CC-B-v14` |
| `feature_name` | varchar(60) | Y | NK part |
| `feature_description` | varchar(500) | Y | |
| `source_table` | varchar(60) | Y | e.g. `fact_revolving_behaviour_monthly` |
| `source_field_or_calc` | varchar(200) | Y | Field name or calculation expression reference |
| `observation_lag_days` | smallint | Y | How many days before `observation_date` this feature is computed as-of — **the leakage-prevention control** |
| `data_type` | varchar(20) | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{feature_definition_sk: 30021, feature_set_version_id: "FS-CC-B-v14", feature_name: "avg_utilization_3m", feature_description: "3-month average utilization ratio", source_table: "fact_revolving_behaviour_monthly", source_field_or_calc: "AVG(utilization_pct) trailing 3 snapshot months", observation_lag_days: 0, data_type: "decimal"}`
**Downstream agents:** Monitoring Agent (feature drift/CSI), Investigation Agent.

## `fact_feature_lineage`
**Business purpose:** row-level (well, batch-level) lineage from a feature value used in a score event back to the source table rows and pipeline run that produced it — the mechanism that makes "why did this feature drift" answerable.
**Grain:** one row per feature per score event.
**Primary Key:** `feature_lineage_sk`
**Foreign Keys:** `score_event_id` → `fact_model_score_event`; `feature_definition_sk` → `dim_feature_definition`
**Natural Key:** (`score_event_id`, `feature_name`)
**SCD Type:** N/A (fact, append-only)
**Source System:** Feature computation pipeline
**Retention:** 2 years (operational lineage; long-tail governance evidence retained via the evidence ledger citation, not this raw table)
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `feature_lineage_sk` | bigint | Y | PK |
| `score_event_id` | varchar(40) | Y | FK |
| `feature_definition_sk` | bigint | Y | FK |
| `feature_value` | decimal(18,6) | N | Null permitted — missingness itself is meaningful |
| `is_missing` | boolean | Y | |
| `source_row_reference` | varchar(200) | Y | e.g. `fact_bureau_aggregate_monthly.bureau_aggregate_sk=71029` |
| `pipeline_run_id` | varchar(30) | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{feature_lineage_sk: 990213, score_event_id: "SCORE-CCB-20260630-ACC0091823", feature_definition_sk: 30021, feature_value: 0.494, is_missing: false, source_row_reference: "fact_revolving_behaviour_monthly.revolving_behaviour_sk=771290", pipeline_run_id: "RUN-2026-06-30-001"}`
**Downstream agents:** Investigation Agent (evidence tracing to root cause), Assurance/Critic Agent (citation verification).

---

# DOMAIN 14 — Model score events

## `fact_model_score_event`
**Business purpose:** the canonical record of a model producing a score for an account — shared grain across A/B/C Score (see `DOMAIN_MODEL.md` §3).
**Grain:** one row per model version + account + score date (+ delinquency bucket for C Score).
**Primary Key:** `score_event_id`
**Foreign Keys:** `model_version_id` → `dim_model_version`; `account_sk` → `dim_account`; `delinquency_bucket_id` → `dim_governance_threshold` (nullable, C Score only)
**Natural Key:** (`model_version_id`, `account_id`, `score_date`, `delinquency_bucket_id`)
**SCD Type:** N/A (immutable fact; a re-score is a new row, never an update — see `TEMPORAL_MODEL.md` on back-scoring)
**Source System:** Scoring pipeline / Azure Function
**Retention:** 7 years
**Sensitivity:** Confidential

| Field | Type | Req | Notes |
|---|---|---|---|
| `score_event_id` | varchar(40) | Y | PK |
| `model_version_id` | varchar(30) | Y | FK |
| `account_sk` | bigint | Y | FK |
| `account_id` | varchar(20) | Y | NK part |
| `observation_date` | date | Y | **Event time — application date (A), snapshot date (B), bucket-observation date (C)** |
| `score_date` | date | Y | Date the score was actually computed (may equal or lag `observation_date`) |
| `mob_at_observation` | smallint | N | Required and ≥6 for B Score; null/0 for A Score; any value for C Score |
| `delinquency_bucket_id` | smallint | N | Required for C Score only |
| `score_value` | decimal(9,6) | Y | |
| `score_band` | varchar(10) | Y | Governed banding, from `dim_governance_threshold` |
| `run_type` | varchar(15) | Y | `live` / `backscore` — **never conflated, per Principle 1.6** |
| `pipeline_run_id` | varchar(30) | Y | |
| `load_timestamp` | timestamp | Y | Processing time |

**Example row:** `{score_event_id: "SCORE-CCB-20260630-ACC0091823", model_version_id: "CC-BSCORE-v2.1", account_id: "ACC-CC-0091823", observation_date: "2026-06-30", score_date: "2026-07-02", mob_at_observation: 7, delinquency_bucket_id: null, score_value: 0.0412, score_band: "B2", run_type: "live"}`
**Downstream agents:** Monitoring Agent, Investigation Agent, Data Quality Agent (MOB-eligibility / version-mismatch checks).

---

# DOMAIN 15 — Performance-label construction

## `fact_performance_label`
**Business purpose:** the forward outcome attached to a score event, with explicit maturity/censoring status — the single most safety-critical table in the schema (ADR-006).
**Grain:** one row per score event per performance window (3/6/12 months).
**Primary Key:** `performance_label_sk`
**Foreign Keys:** `score_event_id` → `fact_model_score_event`
**Natural Key:** (`score_event_id`, `performance_window_months`)
**SCD Type:** N/A (fact; label value can be *restated* as more data arrives, tracked via `label_version`, old version retained not overwritten)
**Source System:** Label construction pipeline
**Retention:** 7 years
**Sensitivity:** Confidential

| Field | Type | Req | Notes |
|---|---|---|---|
| `performance_label_sk` | bigint | Y | PK |
| `score_event_id` | varchar(40) | Y | FK |
| `performance_window_months` | smallint | Y | 3 / 6 / 12 |
| `performance_window_start` | date | Y | = `observation_date` |
| `performance_window_end` | date | Y | = `observation_date + performance_window_months` |
| `outcome_value` | varchar(20) | N | e.g. `good` / `bad` / `cure` / `charge_off` — null if immature |
| `bad_definition_id` | varchar(20) | Y | FK-like reference to the governed bad definition used |
| `maturity_status` | varchar(10) | Y | `mature` / `immature` / `censored` — **hard filter, ADR-006** |
| `censor_reason` | varchar(30) | N | e.g. `account_closed`, `paid_off`, populated only if `censored` |
| `label_version` | smallint | Y | Increments on restatement |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{performance_label_sk: 550210, score_event_id: "SCORE-CCB-20260630-ACC0091823", performance_window_months: 6, performance_window_start: "2026-06-30", performance_window_end: "2026-12-30", outcome_value: null, maturity_status: "immature", label_version: 1}`
**Downstream agents:** Monitoring Agent (mature-population filter is mandatory before any metric calc), Investigation Agent.

---

# DOMAIN 16 — Monitoring metrics

## `fact_monitoring_metric`
**Business purpose:** deterministic output of every population-level monitoring calculation — the table agents read to answer "what is the current PSI." Full metric catalog and formulas in `MONITORING_MART.md` (Part 5).
**Grain:** one row per model version + metric + calculation run + reference/current population definition.
**Primary Key:** `monitoring_metric_sk`
**Foreign Keys:** `model_version_id` → `dim_model_version`
**Natural Key:** (`model_version_id`, `metric_code`, `calculation_run_id`)
**SCD Type:** N/A (fact, append-only per run)
**Source System:** Monitoring pipeline (Azure Function / SQL)
**Retention:** Indefinite (governance record)
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `monitoring_metric_sk` | bigint | Y | PK |
| `model_version_id` | varchar(30) | Y | FK |
| `metric_code` | varchar(20) | Y | e.g. `PSI`, `GINI`, `KS`, `BRIER` |
| `calculation_run_id` | varchar(30) | Y | NK part |
| `run_date` | date | Y | **Monitoring run date (processing time)** |
| `reference_population_def` | varchar(100) | Y | e.g. `development sample` or `prior month` |
| `current_population_def` | varchar(100) | Y | e.g. `2026-06 mature cohort` |
| `metric_value` | decimal(12,6) | Y | |
| `sample_size_current` | int | Y | |
| `minimum_sample_required` | int | Y | From `dim_governance_threshold` |
| `statistical_reliability_flag` | boolean | Y | False if `sample_size_current < minimum_sample_required` |
| `threshold_set_id` | varchar(20) | Y | FK `dim_governance_threshold` |
| `rag_status` | varchar(10) | Y | `green` / `amber` / `red` |
| `calculation_version` | varchar(10) | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{monitoring_metric_sk: 810234, model_version_id: "CC-BSCORE-v2.1", metric_code: "PSI", calculation_run_id: "RUN-MON-2026-06", run_date: "2026-07-05", reference_population_def: "development sample (2024-01)", current_population_def: "2026-06 scored population", metric_value: 0.263, sample_size_current: 11402, rag_status: "amber", calculation_version: "v2"}`
**Downstream agents:** Monitoring Agent (produces), Investigation Agent (consumes as trigger evidence), Reporting & Approval Agent.

---

# DOMAIN 17 — Segment monitoring

## `dim_monitoring_segment`
**Business purpose:** governed catalog of segment definitions (channel, campaign, thin-file flag, geography) so segment cuts are consistent across every monitoring run and every investigation.
**Grain:** one row per segment definition.
**Primary Key:** `segment_id`
**Foreign Keys:** none
**Natural Key:** `segment_id`
**SCD Type:** 1 (definitions corrected in place; rare)
**Source System:** Governance-maintained (Dataverse)
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `segment_id` | varchar(20) | Y | PK |
| `segment_dimension` | varchar(30) | Y | e.g. `acquisition_channel`, `thin_file_flag`, `province` |
| `segment_value` | varchar(50) | Y | e.g. `campaign_partner`, `true` |
| `segment_description` | varchar(200) | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{segment_id: "SEG-THINFILE-TRUE", segment_dimension: "thin_file_flag", segment_value: "true", segment_description: "Customer has fewer than 2 active bureau tradelines at scoring"}`
**Downstream agents:** Monitoring Agent, Investigation Agent.

## `fact_segment_monitoring_metric`
**Business purpose:** metric values computed at segment grain, not population grain — required to detect a mix-shift-driven breach (Scenario 001 cause 1) that a population-level metric alone would not distinguish from a genuine model-quality problem.
**Grain:** one row per model version + metric + calculation run + segment.
**Primary Key:** `segment_monitoring_metric_sk`
**Foreign Keys:** `model_version_id` → `dim_model_version`; `segment_id` → `dim_monitoring_segment`
**Natural Key:** (`model_version_id`, `metric_code`, `calculation_run_id`, `segment_id`)
**SCD Type:** N/A (fact)
**Source System:** Monitoring pipeline
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `segment_monitoring_metric_sk` | bigint | Y | PK |
| `model_version_id` | varchar(30) | Y | FK |
| `metric_code` | varchar(20) | Y | |
| `calculation_run_id` | varchar(30) | Y | |
| `segment_id` | varchar(20) | Y | FK |
| `metric_value` | decimal(12,6) | Y | |
| `population_share_pct` | decimal(6,3) | Y | This segment's share of the current population — the direct evidence for a mix-shift hypothesis |
| `sample_size_current` | int | Y | |
| `statistical_reliability_flag` | boolean | Y | |
| `rag_status` | varchar(10) | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{segment_monitoring_metric_sk: 900112, model_version_id: "CC-BSCORE-v2.1", metric_code: "population_share", calculation_run_id: "RUN-MON-2026-06", segment_id: "SEG-THINFILE-TRUE", metric_value: null, population_share_pct: 0.184, sample_size_current: 2098, rag_status: "amber"}`
**Downstream agents:** Investigation Agent (primary consumer for mix-shift hypothesis testing).

---

# DOMAIN 18 — Data-quality monitoring

## `fact_data_quality_check`
**Business purpose:** deterministic pass/fail/warn results of DQ rules (completeness, missingness, referential integrity, pipeline latency) — the evidence source for Scenario 001 cause 2 and for `DATA_QUALITY_RULES.md`.
**Grain:** one row per DQ rule per table per run.
**Primary Key:** `dq_check_sk`
**Foreign Keys:** none (references table/rule by code)
**Natural Key:** (`rule_code`, `target_table`, `run_id`)
**SCD Type:** N/A (fact)
**Source System:** DQ pipeline
**Retention:** 2 years
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `dq_check_sk` | bigint | Y | PK |
| `rule_code` | varchar(20) | Y | e.g. `DQ-014` |
| `target_table` | varchar(60) | Y | |
| `run_id` | varchar(30) | Y | |
| `run_date` | date | Y | |
| `check_type` | varchar(30) | Y | `completeness` / `missingness` / `referential` / `latency` / `applicability` |
| `result_value` | decimal(12,6) | N | e.g. missingness pct |
| `result_status` | varchar(10) | Y | `pass` / `warn` / `fail` |
| `affected_row_count` | int | N | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{dq_check_sk: 331021, rule_code: "DQ-014", target_table: "fact_bureau_aggregate_monthly", run_id: "RUN-DQ-2026-06", run_date: "2026-07-01", check_type: "missingness", result_value: 0.42, result_status: "fail", affected_row_count: 4788}`
**Downstream agents:** Data Quality Agent (produces), Investigation Agent (consumes as evidence).

---

# DOMAIN 19 — Governance thresholds

## `dim_governance_threshold`
**Business purpose:** policy-as-code home for every threshold, RAG band, eligibility rule and bucket definition — versioned and clause-linked (ADR-007).
**Grain:** one row per threshold rule per version.
**Primary Key:** `threshold_id`
**Foreign Keys:** `policy_clause_id` → `dim_policy_clause`
**Natural Key:** (`threshold_code`, `effective_start_date`)
**SCD Type:** 2
**Source System:** Governance-maintained (Dataverse)
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `threshold_id` | varchar(20) | Y | PK |
| `threshold_code` | varchar(30) | Y | NK part, e.g. `PSI_RAG_CC_BSCORE` |
| `applies_to_model_id` | varchar(20) | N | Null = applies broadly |
| `rule_type` | varchar(20) | Y | `rag_band` / `eligibility` / `bucket_definition` / `escalation` |
| `green_upper_bound` | decimal(9,6) | N | |
| `amber_upper_bound` | decimal(9,6) | N | |
| `minimum_sample_size` | int | N | |
| `eligibility_expression` | varchar(200) | N | e.g. `mob >= 6 AND account_status = 'active'` |
| `policy_clause_id` | varchar(20) | Y | FK |
| `approved_by` | varchar(100) | Y | |
| `effective_start_date` | date | Y | |
| `effective_end_date` | date | N | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{threshold_id: "THR-0091", threshold_code: "PSI_RAG_CC_BSCORE", applies_to_model_id: "CC-BSCORE", rule_type: "rag_band", green_upper_bound: 0.10, amber_upper_bound: 0.25, minimum_sample_size: 1000, policy_clause_id: "POL-MMP-3.2", approved_by: "Model Risk Committee", effective_start_date: "2025-01-01"}`
**Downstream agents:** Monitoring Agent, Governance Agent, Investigation Agent.

---

# DOMAIN 20 — Policy documents and clauses

## `dim_policy_document`
**Business purpose:** governed catalog of policy documents (Model Monitoring Policy, Escalation Matrix, etc.), each versioned, each indexed into Azure AI Search for grounded citation.
**Grain:** one row per policy document per version.
**Primary Key:** `policy_document_id`
**Foreign Keys:** none
**Natural Key:** (`document_code`, `version_label`)
**SCD Type:** 0 (immutable per version)
**Source System:** SharePoint (source), catalogued in Dataverse
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `policy_document_id` | varchar(20) | Y | PK |
| `document_code` | varchar(30) | Y | NK part, e.g. `MMP` |
| `document_title` | varchar(200) | Y | |
| `version_label` | varchar(10) | Y | |
| `sharepoint_uri` | varchar(300) | Y | |
| `effective_date` | date | Y | **Policy effective date** |
| `approved_by` | varchar(100) | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{policy_document_id: "PD-001", document_code: "MMP", document_title: "Model Monitoring Policy", version_label: "v4", sharepoint_uri: "sharepoint.com/.../MMP_v4.pdf", effective_date: "2025-01-01", approved_by: "Model Risk Committee"}`
**Downstream agents:** Governance Agent, Investigation Agent (citation).

## `dim_policy_clause`
**Business purpose:** clause-level granularity so an agent citation points to "MMP v4 §3.2," not just "the policy" — required for Principle 1.4/1.5 to be meaningfully auditable.
**Grain:** one row per clause per document version.
**Primary Key:** `policy_clause_id`
**Foreign Keys:** `policy_document_id` → `dim_policy_document`
**Natural Key:** (`policy_document_id`, `clause_number`)
**SCD Type:** 0
**Source System:** Governance-maintained
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `policy_clause_id` | varchar(20) | Y | PK, e.g. `POL-MMP-3.2` |
| `policy_document_id` | varchar(20) | Y | FK |
| `clause_number` | varchar(10) | Y | `3.2` |
| `clause_text` | varchar(2000) | Y | Full text, indexed to Azure AI Search |
| `clause_topic` | varchar(50) | Y | e.g. `psi_escalation` |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{policy_clause_id: "POL-MMP-3.2", policy_document_id: "PD-001", clause_number: "3.2", clause_text: "A PSI value between 0.10 and 0.25 shall be classified Amber and shall trigger an investigation within 5 business days...", clause_topic: "psi_escalation"}`
**Downstream agents:** Governance Agent (produces citations), Investigation Agent, Assurance/Critic Agent (verifies citation accuracy).

---

# DOMAIN 21 — Investigations

## `fact_investigation`
**Business purpose:** the case record — one row per investigation, from trigger to closure. Central table of the entire governance layer.
**Grain:** one row per investigation.
**Primary Key:** `investigation_id`
**Foreign Keys:** `model_version_id` → `dim_model_version`; `trigger_metric_sk` → `fact_monitoring_metric`; `decision_id` → `fact_committee_decision` (nullable until closed)
**Natural Key:** `investigation_id` (generated)
**SCD Type:** N/A (fact with status field; status transitions logged, not overwritten silently — see `fact_investigation_status_history` conceptually folded into `fact_agent_execution_log` for MVP)
**Source System:** Dataverse (agent-written)
**Retention:** Indefinite (governance record)
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `investigation_id` | varchar(30) | Y | PK |
| `model_version_id` | varchar(30) | Y | FK |
| `trigger_type` | varchar(30) | Y | `metric_breach` / `dq_failure` / `manual` |
| `trigger_metric_sk` | bigint | N | FK, if `trigger_type = metric_breach` |
| `trigger_description` | varchar(300) | Y | |
| `opened_date` | date | Y | **Investigation opened date** |
| `opened_by_agent_id` | varchar(20) | Y | FK `dim_agent` |
| `severity` | varchar(10) | Y | `low` / `medium` / `high` / `critical` |
| `status` | varchar(20) | Y | `open` / `pending_review` / `pending_committee` / `closed` / `escalated` |
| `closed_date` | date | N | **Investigation closed date** |
| `decision_id` | varchar(30) | N | FK, required non-null before `status = closed` (ADR-005) |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{investigation_id: "INV-2026-0031", model_version_id: "CC-BSCORE-v2.1", trigger_type: "metric_breach", trigger_metric_sk: 810234, trigger_description: "PSI amber (0.263) for CC Behaviour Score, June 2026 run", opened_date: "2026-07-05", opened_by_agent_id: "AGT-MONITORING", severity: "medium", status: "closed", closed_date: "2026-07-22", decision_id: "DEC-2026-0014"}`
**Downstream agents:** Investigation Agent, Assurance/Critic Agent, Reporting & Approval Agent.

---

# DOMAIN 22 — Evidence ledger

## `fact_evidence_item`
**Business purpose:** the single structure for every citable fact any agent uses (ADR-008). Full worked content in `demo/SAMPLE_EVIDENCE_LEDGER.md`.
**Grain:** one row per discrete evidence fact.
**Primary Key:** `evidence_item_id`
**Foreign Keys:** `investigation_id` → `fact_investigation`; `collected_by_agent_id` → `dim_agent`; `policy_clause_id` → `dim_policy_clause` (nullable)
**Natural Key:** `evidence_item_id` (generated)
**SCD Type:** N/A (immutable once written — evidence is never edited, only superseded by a new item with `supersedes_evidence_item_id`)
**Source System:** Dataverse (agent-written)
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `evidence_item_id` | varchar(30) | Y | PK |
| `investigation_id` | varchar(30) | Y | FK |
| `evidence_type` | varchar(20) | Y | `metric_value` / `table_row` / `tool_output` / `policy_clause` / `dq_check` |
| `source_table` | varchar(60) | N | Populated for `metric_value`/`table_row`/`dq_check` types |
| `source_row_identifier` | varchar(100) | N | e.g. `fact_monitoring_metric.monitoring_metric_sk=810234` |
| `tool_execution_id` | varchar(30) | N | FK `fact_tool_execution_log`, populated for `tool_output` type |
| `policy_clause_id` | varchar(20) | N | FK, populated for `policy_clause` type |
| `evidence_value_summary` | varchar(500) | Y | Human-readable summary of the fact |
| `collected_by_agent_id` | varchar(20) | Y | FK |
| `collected_timestamp` | timestamp | Y | |
| `conclusion_supported` | varchar(200) | Y | Which hypothesis/finding this evidence bears on |
| `confidence` | varchar(10) | Y | `high` / `medium` / `low` |
| `is_contradictory` | boolean | Y | True if this evidence argues *against* the leading hypothesis — required to exist for a well-formed investigation (Assurance Agent checks for at least an attempt at counter-evidence) |
| `supersedes_evidence_item_id` | varchar(30) | N | Self-FK |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{evidence_item_id: "EVID-2026-0031-04", investigation_id: "INV-2026-0031", evidence_type: "table_row", source_table: "fact_segment_monitoring_metric", source_row_identifier: "segment_monitoring_metric_sk=900112", evidence_value_summary: "Thin-file segment share of scored population rose from 8.1% (May) to 18.4% (June)", collected_by_agent_id: "AGT-INVESTIGATION", conclusion_supported: "H1: campaign-driven mix shift", confidence: "high", is_contradictory: false}`
**Downstream agents:** Investigation Agent (produces), Assurance/Critic Agent (verifies), Reporting & Approval Agent (cites in report).

## `fact_finding_evidence_link`
**Business purpose:** many-to-many bridge so a finding cites multiple evidence items and an evidence item can support multiple findings, without duplicating evidence rows.
**Grain:** one row per (finding, evidence item) pair.
**Primary Key:** (`finding_id`, `evidence_item_id`) composite
**Foreign Keys:** `finding_id` → `fact_finding`; `evidence_item_id` → `fact_evidence_item`
**Natural Key:** same as PK
**SCD Type:** N/A
**Source System:** Dataverse
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `finding_id` | varchar(30) | Y | PK/FK |
| `evidence_item_id` | varchar(30) | Y | PK/FK |
| `link_role` | varchar(20) | Y | `supporting` / `contradictory` |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{finding_id: "FIND-2026-0031-01", evidence_item_id: "EVID-2026-0031-04", link_role: "supporting"}`
**Downstream agents:** Assurance/Critic Agent, Reporting & Approval Agent.

---

# DOMAIN 23 — Findings and root causes

## `fact_finding`
**Business purpose:** a conclusion the Investigation Agent believes the evidence supports, pending critic review.
**Grain:** one row per finding.
**Primary Key:** `finding_id`
**Foreign Keys:** `investigation_id` → `fact_investigation`; `root_cause_code` → `dim_root_cause_taxonomy`
**Natural Key:** `finding_id`
**SCD Type:** N/A (immutable; a revised finding is a new row with `revises_finding_id`)
**Source System:** Dataverse (agent-written)
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `finding_id` | varchar(30) | Y | PK |
| `investigation_id` | varchar(30) | Y | FK |
| `finding_statement` | varchar(1000) | Y | |
| `root_cause_code` | varchar(30) | Y | FK, controlled taxonomy — never free text |
| `severity` | varchar(10) | Y | |
| `critic_review_status` | varchar(20) | Y | `pending` / `approved` / `rejected` |
| `critic_review_notes` | varchar(1000) | N | |
| `revises_finding_id` | varchar(30) | N | Self-FK |
| `created_by_agent_id` | varchar(20) | Y | FK |
| `created_timestamp` | timestamp | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{finding_id: "FIND-2026-0031-01", investigation_id: "INV-2026-0031", finding_statement: "The June PSI Amber breach is primarily attributable to a mix shift toward thin-file customers following campaign CMP-2025-THINFILE-04, compounded by bureau feature missingness.", root_cause_code: "RC-POPULATION-MIX-SHIFT", severity: "medium", critic_review_status: "approved", created_by_agent_id: "AGT-INVESTIGATION"}`
**Downstream agents:** Assurance/Critic Agent, Reporting & Approval Agent.

## `dim_root_cause_taxonomy`
**Business purpose:** controlled vocabulary of root cause categories — prevents free-text root causes that can't be aggregated or trended across investigations.
**Grain:** one row per root cause code.
**Primary Key:** `root_cause_code`
**Foreign Keys:** none
**Natural Key:** `root_cause_code`
**SCD Type:** 1
**Source System:** Governance-maintained
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `root_cause_code` | varchar(30) | Y | PK, e.g. `RC-POPULATION-MIX-SHIFT` |
| `root_cause_category` | varchar(30) | Y | `population` / `data_quality` / `policy` / `model` / `external` |
| `root_cause_description` | varchar(300) | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{root_cause_code: "RC-POPULATION-MIX-SHIFT", root_cause_category: "population", root_cause_description: "Scored population composition shifted materially versus the reference/development population"}`
**Downstream agents:** Investigation Agent, Reporting & Approval Agent (trending root causes across investigations over time).

## `fact_finding_contributing_factor`
**Business purpose:** captures that a finding can have more than one contributing factor (Scenario 001 has three linked causes) without conflating them into a single root cause field.
**Grain:** one row per (finding, contributing factor) pair.
**Primary Key:** `finding_contributing_factor_sk`
**Foreign Keys:** `finding_id` → `fact_finding`; `root_cause_code` → `dim_root_cause_taxonomy`
**Natural Key:** (`finding_id`, `root_cause_code`)
**SCD Type:** N/A
**Source System:** Dataverse
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `finding_contributing_factor_sk` | bigint | Y | PK |
| `finding_id` | varchar(30) | Y | FK |
| `root_cause_code` | varchar(30) | Y | FK |
| `contribution_weight` | varchar(10) | Y | `primary` / `secondary` / `minor` |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{finding_contributing_factor_sk: 12031, finding_id: "FIND-2026-0031-01", root_cause_code: "RC-BUREAU-DATA-MISSINGNESS", contribution_weight: "secondary"}`
**Downstream agents:** Investigation Agent, Reporting & Approval Agent.

---

# DOMAIN 24 — Recommendations

## `fact_recommendation`
**Business purpose:** the proposed action arising from a finding, always requiring human approval before any downstream action is taken.
**Grain:** one row per recommendation.
**Primary Key:** `recommendation_id`
**Foreign Keys:** `finding_id` → `fact_finding`; `investigation_id` → `fact_investigation`
**Natural Key:** `recommendation_id`
**SCD Type:** N/A
**Source System:** Dataverse (agent-written)
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `recommendation_id` | varchar(30) | Y | PK |
| `finding_id` | varchar(30) | Y | FK |
| `investigation_id` | varchar(30) | Y | FK |
| `recommendation_type` | varchar(30) | Y | `monitor` / `investigate_further` / `recalibration` / `feature_fix` / `policy_review` / `no_action` |
| `recommendation_text` | varchar(1000) | Y | |
| `proposed_owner` | varchar(100) | Y | Accountable business owner if approved |
| `requires_committee_approval` | boolean | Y | True by default for anything above `monitor` (ADR-005/Escalation Matrix) |
| `status` | varchar(20) | Y | `proposed` / `approved` / `rejected` / `superseded` |
| `created_by_agent_id` | varchar(20) | Y | FK |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{recommendation_id: "REC-2026-0031-01", finding_id: "FIND-2026-0031-01", investigation_id: "INV-2026-0031", recommendation_type: "monitor", recommendation_text: "Continue monthly monitoring at current cadence; add a standing segment-level PSI cut for thin-file customers; open a data-quality remediation ticket for bureau feed BRU-FEED-07.", proposed_owner: "Retail Risk Analytics", requires_committee_approval: true, status: "approved"}`
**Downstream agents:** Reporting & Approval Agent, Assurance/Critic Agent.

---

# DOMAIN 25 — Human approvals and committee decisions

## `fact_committee_decision`
**Business purpose:** the formal record of a Model Risk Committee ruling on an investigation/recommendation — the terminal governance event.
**Grain:** one row per committee decision.
**Primary Key:** `decision_id`
**Foreign Keys:** `investigation_id` → `fact_investigation`; `recommendation_id` → `fact_recommendation`
**Natural Key:** `decision_id`
**SCD Type:** N/A (immutable)
**Source System:** Dataverse (human-entered via Reporting & Approval Agent's Teams approval card)
**Retention:** Indefinite (permanent governance record)
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `decision_id` | varchar(30) | Y | PK |
| `investigation_id` | varchar(30) | Y | FK |
| `recommendation_id` | varchar(30) | N | FK, null if decision is a rejection with no specific recommendation actioned |
| `decision` | varchar(20) | Y | `approve` / `reject` / `request_more_evidence` |
| `decision_rationale` | varchar(1000) | Y | |
| `decided_by` | varchar(100) | Y | Named Entra identity |
| `committee_name` | varchar(50) | Y | e.g. `Model Risk Committee` |
| `decision_date` | date | Y | **Committee decision date** |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{decision_id: "DEC-2026-0014", investigation_id: "INV-2026-0031", recommendation_id: "REC-2026-0031-01", decision: "approve", decision_rationale: "Evidence trace supports mix-shift and bureau-missingness explanation; PSI expected to normalize as campaign cohort matures. Approve recommendation as written; require DQ ticket closure confirmation within 30 days.", decided_by: "J. Sirisak (MRC Chair)", committee_name: "Model Risk Committee", decision_date: "2026-07-22"}`
**Downstream agents:** Reporting & Approval Agent (produces the approval card and records the outcome).

## `fact_human_approval`
**Business purpose:** lighter-weight approval record for pre-committee steps (analyst sign-off that a finding is ready for committee) — distinct from a full committee decision.
**Grain:** one row per approval action.
**Primary Key:** `approval_id`
**Foreign Keys:** `investigation_id` → `fact_investigation`; `finding_id` → `fact_finding` (nullable)
**Natural Key:** `approval_id`
**SCD Type:** N/A
**Source System:** Dataverse
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `approval_id` | varchar(30) | Y | PK |
| `investigation_id` | varchar(30) | Y | FK |
| `finding_id` | varchar(30) | N | FK |
| `approval_stage` | varchar(30) | Y | `analyst_review` / `pre_committee_sign_off` |
| `approved_by` | varchar(100) | Y | |
| `approval_outcome` | varchar(20) | Y | `approved` / `sent_back` |
| `approval_timestamp` | timestamp | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{approval_id: "APR-2026-0031-01", investigation_id: "INV-2026-0031", finding_id: "FIND-2026-0031-01", approval_stage: "analyst_review", approved_by: "N. Thanakit (Model Risk Analyst)", approval_outcome: "approved", approval_timestamp: "2026-07-18T09:12:00Z"}`
**Downstream agents:** Reporting & Approval Agent.

---

# DOMAIN 26 — Remediation actions

## `fact_remediation_action`
**Business purpose:** tracks an approved recommendation through to completion — recorded, never executed, by any agent (Principle 1.10).
**Grain:** one row per remediation action.
**Primary Key:** `remediation_action_id`
**Foreign Keys:** `recommendation_id` → `fact_recommendation`; `decision_id` → `fact_committee_decision`
**Natural Key:** `remediation_action_id`
**SCD Type:** N/A (status field tracks progress)
**Source System:** Dataverse (updated by the accountable owner, not by an agent)
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `remediation_action_id` | varchar(30) | Y | PK |
| `recommendation_id` | varchar(30) | Y | FK |
| `decision_id` | varchar(30) | Y | FK |
| `action_description` | varchar(500) | Y | |
| `accountable_owner` | varchar(100) | Y | |
| `target_completion_date` | date | Y | |
| `actual_completion_date` | date | N | |
| `status` | varchar(20) | Y | `open` / `in_progress` / `complete` / `overdue` |
| `closure_evidence_item_id` | varchar(30) | N | FK `fact_evidence_item`, populated on completion |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{remediation_action_id: "REM-2026-0031-01", recommendation_id: "REC-2026-0031-01", decision_id: "DEC-2026-0014", action_description: "Bureau data engineering to fix upstream feed BRU-FEED-07 missingness spike", accountable_owner: "Data Engineering — Bureau Integration", target_completion_date: "2026-08-21", status: "in_progress"}`
**Downstream agents:** Reporting & Approval Agent (status tracking, follow-up reminders).

---

# DOMAIN 27 — Agent inventory

## `dim_agent`
**Business purpose:** registry of every agent, its version, and its declared boundaries — the row every execution log entry links back to.
**Grain:** one row per agent per version.
**Primary Key:** `agent_id` composite with version, or `agent_version_sk` surrogate
**Foreign Keys:** `prompt_version_id` → `dim_prompt_version`
**Natural Key:** (`agent_code`, `agent_version`)
**SCD Type:** 0 (immutable per version)
**Source System:** Dataverse (build-time registration)
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `agent_version_sk` | bigint | Y | PK |
| `agent_id` | varchar(20) | Y | NK part, e.g. `AGT-INVESTIGATION` |
| `agent_name` | varchar(60) | Y | |
| `agent_version` | varchar(10) | Y | NK part |
| `agent_role` | varchar(200) | Y | Short role statement |
| `prompt_version_id` | varchar(20) | Y | FK |
| `allowed_data_domains` | varchar(500) | Y | Documented list, enforced separately at connector layer |
| `allowed_tools` | varchar(500) | Y | |
| `deployed_date` | date | Y | |
| `status` | varchar(15) | Y | `active` / `retired` |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{agent_version_sk: 7, agent_id: "AGT-INVESTIGATION", agent_name: "Investigation Agent", agent_version: "v1.2", agent_role: "Investigates monitoring/DQ triggers into evidence-grounded findings and recommendations", prompt_version_id: "PV-INV-1.2", status: "active"}`
**Downstream agents:** referenced by all execution/evaluation logging tables.

---

# DOMAIN 28 — Agent execution logs

## `fact_agent_execution_log`
**Business purpose:** full audit trail of every agent invocation — see `architecture/OBSERVABILITY.md` for field-by-field logging rationale.
**Grain:** one row per agent invocation.
**Primary Key:** `execution_id`
**Foreign Keys:** `agent_version_sk` → `dim_agent`; `prompt_version_id` → `dim_prompt_version`
**Natural Key:** `execution_id`
**SCD Type:** N/A
**Source System:** Copilot Studio (logged via Power Automate/Dataverse action on every turn)
**Retention:** 2 years rolling
**Sensitivity:** Internal (payloads redacted per sensitivity rules before logging)

| Field | Type | Req | Notes |
|---|---|---|---|
| `execution_id` | varchar(40) | Y | PK |
| `correlation_id` | varchar(40) | Y | Threads the whole request chain |
| `agent_version_sk` | bigint | Y | FK |
| `prompt_version_id` | varchar(20) | Y | FK |
| `invocation_time` | timestamp | Y | Event time |
| `logged_time` | timestamp | Y | Processing time |
| `input_payload` | varchar(max) | Y | JSON, redacted |
| `output_payload` | varchar(max) | Y | JSON |
| `status` | varchar(15) | Y | `success` / `error` / `escalated` |
| `error_detail` | varchar(1000) | N | |
| `duration_ms` | int | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{execution_id: "EXEC-99213", correlation_id: "CORR-2026-0031", agent_version_sk: 7, prompt_version_id: "PV-INV-1.2", invocation_time: "2026-07-05T10:02:11Z", status: "success", duration_ms: 4210}`
**Downstream agents:** Assurance/Critic Agent (auditability checks), human audit review.

---

# DOMAIN 29 — Tool execution logs

## `fact_tool_execution_log`
**Business purpose:** every tool/action call an agent makes, linked back to the triggering agent execution.
**Grain:** one row per tool call.
**Primary Key:** `tool_execution_id`
**Foreign Keys:** `execution_id` → `fact_agent_execution_log`; `tool_id` → `dim_tool` (see `tools/TOOL_CATALOG.md`)
**Natural Key:** `tool_execution_id`
**SCD Type:** N/A
**Source System:** Copilot Studio action wrapper
**Retention:** 2 years rolling
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `tool_execution_id` | varchar(40) | Y | PK |
| `correlation_id` | varchar(40) | Y | |
| `execution_id` | varchar(40) | Y | FK |
| `tool_id` | varchar(20) | Y | FK |
| `request_payload` | varchar(max) | Y | JSON |
| `response_payload` | varchar(max) | Y | JSON |
| `status` | varchar(15) | Y | `success` / `error` / `timeout` |
| `duration_ms` | int | Y | |
| `invocation_time` | timestamp | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{tool_execution_id: "TEXEC-441209", correlation_id: "CORR-2026-0031", execution_id: "EXEC-99213", tool_id: "TOOL-GET-SEGMENT-METRIC", status: "success", duration_ms: 312}`
**Downstream agents:** Assurance/Critic Agent, evidence-lineage citation verification.

---

# DOMAIN 30 — Prompt and instruction versions

## `dim_prompt_version`
**Business purpose:** version history of every agent's instruction set, so a behaviour change can be traced to a specific prompt edit — the direct implementation of Principle 1.6 for agents.
**Grain:** one row per prompt version.
**Primary Key:** `prompt_version_id`
**Foreign Keys:** none
**Natural Key:** (`agent_id`, `version_label`)
**SCD Type:** 0 (immutable)
**Source System:** Git-backed `prompts/` folder, registered in Dataverse on deploy
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `prompt_version_id` | varchar(20) | Y | PK |
| `agent_id` | varchar(20) | Y | NK part |
| `version_label` | varchar(10) | Y | NK part |
| `prompt_file_path` | varchar(200) | Y | e.g. `prompts/INVESTIGATION_INSTRUCTIONS.md` |
| `change_summary` | varchar(500) | Y | |
| `approved_by` | varchar(100) | Y | |
| `effective_date` | date | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{prompt_version_id: "PV-INV-1.2", agent_id: "AGT-INVESTIGATION", version_label: "1.2", prompt_file_path: "prompts/INVESTIGATION_INSTRUCTIONS.md", change_summary: "Added mandatory counter-evidence search step", approved_by: "Copilot Studio Solution Architect", effective_date: "2026-06-01"}`
**Downstream agents:** referenced by `dim_agent`, `fact_agent_execution_log`.

---

# DOMAIN 31 — Knowledge-source versions

## `dim_knowledge_source_version`
**Business purpose:** version tracking for the Azure AI Search index content (policy documents, root-cause taxonomy) so a grounding citation can be tied to the exact index snapshot used.
**Grain:** one row per knowledge source per index build.
**Primary Key:** `knowledge_source_version_id`
**Foreign Keys:** `policy_document_id` → `dim_policy_document` (nullable, for non-policy sources)
**Natural Key:** (`source_code`, `index_build_id`)
**SCD Type:** 0
**Source System:** Azure AI Search indexing pipeline
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `knowledge_source_version_id` | varchar(20) | Y | PK |
| `source_code` | varchar(30) | Y | e.g. `POLICY_INDEX` |
| `index_build_id` | varchar(30) | Y | |
| `document_count_indexed` | int | Y | |
| `index_build_timestamp` | timestamp | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{knowledge_source_version_id: "KSV-0044", source_code: "POLICY_INDEX", index_build_id: "IDX-2026-06-30", document_count_indexed: 6, index_build_timestamp: "2026-06-30T23:00:00Z"}`
**Downstream agents:** Governance Agent, Investigation Agent (know which index snapshot grounded a given citation).

---

# DOMAIN 32 — Agent evaluations and override monitoring

## `fact_agent_evaluation`
**Business purpose:** structured record of critic review outcomes and periodic agent-quality evaluation (not just pass/fail per finding, but trended evaluation against the rubrics in `agents/AGENT_EVALUATION.md`).
**Grain:** one row per evaluation event.
**Primary Key:** `evaluation_id`
**Foreign Keys:** `agent_version_sk` → `dim_agent`; `execution_id` → `fact_agent_execution_log` (nullable, for finding-level evaluation)
**Natural Key:** `evaluation_id`
**SCD Type:** N/A
**Source System:** Assurance/Critic Agent (per-finding), manual test harness (periodic batch eval)
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `evaluation_id` | varchar(30) | Y | PK |
| `agent_version_sk` | bigint | Y | FK |
| `execution_id` | varchar(40) | N | FK |
| `evaluation_type` | varchar(20) | Y | `critic_review` / `batch_eval` |
| `rubric_score` | decimal(5,2) | N | 0–100, batch eval only |
| `grounding_pass` | boolean | Y | |
| `citation_pass` | boolean | Y | |
| `outcome` | varchar(20) | Y | `pass` / `fail` / `conditional_pass` |
| `evaluator` | varchar(50) | Y | `AGT-ASSURANCE` or human evaluator name |
| `evaluation_timestamp` | timestamp | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** `{evaluation_id: "EVAL-2026-0031-01", agent_version_sk: 7, execution_id: "EXEC-99213", evaluation_type: "critic_review", grounding_pass: true, citation_pass: true, outcome: "pass", evaluator: "AGT-ASSURANCE", evaluation_timestamp: "2026-07-18T08:55:00Z"}`
**Downstream agents:** Assurance/Critic Agent (produces), Reporting & Approval Agent (quality trend in committee report appendix).

## `fact_agent_override`
**Business purpose:** every time a human overrides or rejects an agent conclusion — the leading indicator that an agent is drifting or a taxonomy/threshold needs review.
**Grain:** one row per override event.
**Primary Key:** `override_id`
**Foreign Keys:** `finding_id` → `fact_finding` (nullable); `recommendation_id` → `fact_recommendation` (nullable); `decision_id` → `fact_committee_decision`
**Natural Key:** `override_id`
**SCD Type:** N/A
**Source System:** Dataverse (recorded whenever a committee decision disagrees with an agent recommendation)
**Retention:** Indefinite
**Sensitivity:** Internal

| Field | Type | Req | Notes |
|---|---|---|---|
| `override_id` | varchar(30) | Y | PK |
| `finding_id` | varchar(30) | N | FK |
| `recommendation_id` | varchar(30) | N | FK |
| `decision_id` | varchar(30) | Y | FK |
| `override_reason` | varchar(500) | Y | |
| `override_category` | varchar(30) | Y | `agent_error` / `business_context_agent_lacked` / `policy_judgment_call` |
| `overridden_by` | varchar(100) | Y | |
| `override_timestamp` | timestamp | Y | |
| `load_timestamp` | timestamp | Y | |

**Example row:** none in the MVP demo (the engineered scenario is designed to be approved as-is — see `scenarios/SCENARIO-001-psi-breach.md`); a hypothetical row: `{override_id: "OVR-2026-0002", recommendation_id: "REC-2026-0018-01", decision_id: "DEC-2026-0009", override_reason: "Committee judged recalibration premature pending one more monitoring cycle", override_category: "policy_judgment_call", overridden_by: "MRC Chair"}`
**Downstream agents:** Reporting & Approval Agent (override-rate trend, feeds `MONITORING_MART.md` "override rate" metric), used to evaluate agent quality over time.

---

# Table count summary

| Layer | Domains | Physical tables |
|---|---|---|
| A — Source of truth | 1–3 | 4 (`dim_customer`, `fact_application`, `dim_account`, + 2 extensions counted once each = `dim_account_revolving_ext`, `dim_account_termloan_ext`) |
| B — Behavioural time series | 4–10 | 7 |
| C — Model and scoring | 11–16 | 9 |
| D — Governance surface | 17–20 | 6 |
| E — Investigation/evidence/agent | 21–32 | 19 |
| **Total** | **32** | **45** |
