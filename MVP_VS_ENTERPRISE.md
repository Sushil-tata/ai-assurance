# MVP versus Enterprise Target (Part 11)

**Audience:** sponsors, architects, Model Risk Committee.
**Rule applied:** every capability below is marked exactly one of **Implemented**, **Mocked**, **Sample data only**, or **Architecture only** — no capability is left ambiguous.

## 1. Products

| Product | MVP | Enterprise |
|---|---|---|
| Credit Card | **Implemented** — full live data, scoring, monitoring | Implemented |
| Speedy Cash | Architecture only — schema supports it (`dim_account_revolving_ext` handles both CC and SPC identically); zero synthetic data generated | Implemented |
| Speedy Loan | Architecture only — `dim_account_termloan_ext` fully specified; zero synthetic data generated | Implemented |

## 2. Model families and models

| Model | MVP | Enterprise |
|---|---|---|
| CC Behaviour Score (B) | **Implemented** — live scoring, monitoring, one full investigation | Implemented |
| CC Application Score (A) | Sample data only — `dim_model`/`dim_model_version` registry row exists; no score events generated | Implemented |
| CC Bucket 0 / Bucket 1 C Score | Sample data only — registry rows exist | Implemented |
| All SPC and SPL models (A/B/C, 8 models) | Architecture only — registry rows exist per `data/MODEL_INVENTORY_SCHEMA.md`, no data at all | Implemented |

## 3. Model lifecycle

| Stage | MVP | Enterprise |
|---|---|---|
| `production_monitoring` | **Implemented** for CC B Score | Implemented for all 12 |
| `business_requirement` → `approval` | Architecture only — lifecycle stages exist as enum values, no workflow tooling built | Implemented, with workflow tooling |
| `recalibration` / `redevelopment` | Architecture only — referenced in Scenario 003/006, no active workflow | Implemented |
| `retirement` | Architecture only | Implemented |

## 4. Monitoring metrics

| Metric group | MVP | Enterprise |
|---|---|---|
| PSI, CSI, missingness, population count | **Implemented** | Implemented |
| Gini, KS, calibration O/E, Brier | **Implemented** (used in Scenario 003 test fixture; Scenario 001 focuses on PSI/CSI) | Implemented |
| Roll rate, cure rate, redefault | Mocked — formula specified in `data/MONITORING_MART.md` §1.17-1.19, not computed against live data since no C Score data exists in MVP | Implemented |
| Segment/vintage/MOB performance | **Implemented** for CC B Score (used directly in Scenario 001) | Implemented across all models |
| Override rate | Sample data only — `fact_agent_override` schema exists, no override events generated in the clean Scenario 001 path (by design) | Implemented, trended over time |

## 5. Investigation and governance

| Capability | MVP | Enterprise |
|---|---|---|
| Full investigation → evidence → finding → recommendation → decision cycle | **Implemented**, Scenario 001 end to end | Implemented, at volume |
| Critic loop | **Implemented** | Implemented |
| Escalation matrix | **Implemented** for the rows exercised by Scenarios 001, 004, 005; other rows architecture-only | Implemented, all rows live-tested |
| Remediation tracking | **Implemented** through `in_progress` status (closure evidence not generated, since demo timeline doesn't reach completion) | Implemented, full lifecycle |

## 6. Agents

| Agent | MVP | Enterprise |
|---|---|---|
| All seven | **Implemented** | Implemented, plus enterprise-only additions considered in `RED_TEAM_REVIEW.md` (e.g., a dedicated Remediation Tracking Agent) — **not built in either MVP or the current enterprise design**, flagged as a future consideration only |

## 7. Microsoft components

| Component | MVP | Enterprise |
|---|---|---|
| Copilot Studio, Dataverse, Power Automate, Teams, Fabric Warehouse, Azure AI Search, Azure Functions | **Implemented** | Implemented, hardened |
| Fabric Lakehouse | **Implemented** (synthetic data curation) | Implemented (real source curation) |
| Power BI | **Implemented** (committee report visuals) | Implemented, broader dashboard suite |
| Purview | Architecture only — documented lineage in `data/TABLE_CATALOG.md`, no automated Purview cataloguing | Implemented |
| Entra ID least privilege | Mocked — manually configured roles per `architecture/SECURITY_ARCHITECTURE.md`, not a hardened production configuration | Implemented, audited |
| Application Insights | **Implemented** (basic auto-instrumentation) | Implemented, with automated alerting |

## 8. Data

| Aspect | MVP | Enterprise |
|---|---|---|
| CC customer/account/behaviour/bureau data | **Implemented**, synthetic | Real, curated |
| SPC/SPL data | Architecture only | Real, curated |
| Data retention/classification enforcement | Mocked — documented view-level exclusion of restricted columns, no Purview-enforced labeling | Implemented |

## 9. What this table proves

Every "architecture only" or "sample data only" row above has a fully specified schema in `data/TABLE_CATALOG.md` and, where relevant, a fully specified model inventory row in `data/MODEL_INVENTORY_SCHEMA.md`. Extending from MVP to enterprise is additive — populate more rows, register more models, wire more scheduled runs — not a redesign. This is the direct payoff of `adr/ADR-009-mvp-product-model.md`'s parent/extension pattern.
