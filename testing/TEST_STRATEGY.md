# Test Strategy

**Audience:** whoever builds and QAs the MVP (likely also the sole builder, per `DELIVERY_PLAN.md`).

## 1. Test layers

| Layer | What it tests | Tooling |
|---|---|---|
| Data contract tests | Synthetic data conforms to `data/TABLE_CATALOG.md` (types, keys, referential integrity, maturity logic) | SQL assertions run against Fabric Warehouse after each synthetic generation batch |
| Deterministic metric tests | Metric pipeline formulas produce correct values on known fixture inputs | Python unit tests against the Azure Function calculation layer, independent of any agent |
| Agent unit tests | Each agent, given a scripted input, stays within its documented boundary and produces schema-conformant output | Scripted conversation transcripts run against the deployed Copilot Studio agent via API |
| Policy tests | Threshold/eligibility logic resolves correctly for known dates/models | SQL assertions against `dim_governance_threshold` |
| End-to-end tests | Full Scenario 001 trace, trigger → committee decision, produces the expected artifacts | Scripted run of the full pipeline against the demo dataset |
| Adversarial tests | Each agent's contract "Adversarial test cases" section | Scripted, run before every prompt-version deploy |

## 2. What is NOT tested in MVP (explicitly)

- Load/performance testing at production data volumes (enterprise-target concern).
- Multi-tenant / multi-environment isolation testing beyond dev/demo separation.
- Penetration testing of the Entra/Copilot Studio configuration (documented as an enterprise pre-production gate, not part of this build).

## 3. Test data

All tests run against the synthetic dataset (`data/SYNTHETIC_DATA_SPEC.md`) plus purpose-built fixture extensions for each scenario in `scenarios/` — Scenario 001's three causes are literally encoded as generation parameters in the synthetic data generator, not hand-edited after the fact, so the test is reproducible from a clean regeneration.

## 4. Definition of "MVP test-complete"
Every item in `ACCEPTANCE_CRITERIA.md` passes against the demo dataset, and the full Scenario 001 trace can be reproduced end-to-end from a clean environment in under the time budgeted in `DELIVERY_PLAN.md` step 13.
