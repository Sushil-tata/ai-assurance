# Scope — AI Assurance MVP and Enterprise Target

## 1. MVP scope (build this first, build this fully)

| Dimension | MVP value |
|---|---|
| Product | Credit Card (revolving) only |
| Model family | B Score (Behaviour Score) only |
| Model | Credit Card Behaviour Score, `model_id = CC-BSCORE-001` |
| Eligibility | MOB ≥ 6, active account, not written off |
| Scoring frequency | Monthly snapshot |
| Performance windows | Forward 3, 6, 12 months from score date |
| Monitoring metrics | PSI, CSI, missingness, Gini, KS, calibration (observed-to-expected), bad rate, population count, sample-size sufficiency — the minimum set that supports one full investigation narrative |
| Investigation | One engineered scenario: Behaviour Score PSI Amber in June (three linked causes) — see `scenarios/SCENARIO-001-psi-breach.md` |
| Agents | All seven: Orchestrator, Monitoring, Data Quality, Governance, Investigation, Assurance/Critic, Reporting & Approval |
| Tools | 4–6: metric query, evidence lookup, policy lookup, investigation CRUD, (optionally) feature-drift query, (optionally) notification |
| Approval flow | One: Investigation → Recommendation → Committee decision (approve / reject / request more evidence) |
| Committee report | One generated report, `demo/SAMPLE_COMMITTEE_REPORT.md` |
| Data | 100% synthetic, internally consistent, `data/SYNTHETIC_DATA_SPEC.md` |

## 2. Explicitly out of MVP scope

- Speedy Cash and Speedy Loan data, scoring or monitoring (schema supports it; data and agents do not populate it in MVP).
- A Score and C Score live monitoring (schema and model inventory rows exist; no monitoring runs are demonstrated).
- Model development, training or challenger-model comparison.
- Any write-back to a production scoring engine.
- Real customer or bureau data of any kind.
- Multi-language support, mobile client, or any UI beyond Teams + Copilot Studio canvas + Power BI report.
- Automated remediation execution (remediation actions are *recorded*, never *executed*, by agents).

## 3. Enterprise target (architecture must support; MVP does not build)

- All three products, all three model families → 12 model inventory rows (see `data/MODEL_INVENTORY_SCHEMA.md`), each independently scheduled and monitored.
- Full model lifecycle (business requirement → retirement) with recalibration and redevelopment triggers.
- Full monitoring metric catalog (Part 5 of the design brief; ~24 metrics) at population, segment and vintage grain.
- Purview-catalogued lineage from raw source to committee report.
- Entra ID–enforced least privilege per agent and per data domain, not just documented boundaries.
- Application Insights distributed tracing across every agent and tool call.
- Broader agent inventory (e.g., a dedicated Remediation Tracking Agent, a Model Change Agent) — deferred, not designed against in this version beyond a placeholder in `RED_TEAM_REVIEW.md`.

## 4. Explicit non-goals (permanent, not just MVP-deferred)

- AI Assurance will never be the system of record for credit decisions.
- AI Assurance will never train or retrain a model.
- AI Assurance agents will never have write access to any production scoring or account system.

## 5. Traceability

Every item above maps to a build step in `DELIVERY_PLAN.md` and a red-team check in `RED_TEAM_REVIEW.md`. If a capability appears in a demo but not in this file, treat it as scope creep and flag it.
