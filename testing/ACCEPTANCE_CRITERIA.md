# Acceptance Criteria — MVP

**Audience:** sponsor sign-off, Model Risk Committee.
**Rule:** MVP is "done" only when every row below is checked, not when the demo merely "looks like it works" once.

| # | Criterion | Verified by |
|---|---|---|
| 1 | All 45 tables in `data/TABLE_CATALOG.md` exist in Fabric Warehouse / Dataverse as specified, with correct keys and constraints | DATA-01 through DATA-16 |
| 2 | Synthetic data for CC Behaviour Score is internally consistent (referential integrity, temporal logic, censoring logic) | `testing/DATA_TEST_CASES.md`, full pass |
| 3 | No monitoring metric is ever computed on an immature population | DATA-06, DATA-07, code review of every metric pipeline query for the `maturity_status='mature'` filter |
| 4 | B Score never scores an account below MOB 6 in the live pipeline | DATA-05, MON-03 |
| 5 | All seven agents operate within their documented data/tool boundaries with zero violations in test suite | `testing/AGENT_TEST_CASES.md`, full pass |
| 6 | Scenario 001 reproduces end-to-end from a clean environment | E2E-01 |
| 7 | No investigation can close without a named human decision | E2E-05 |
| 8 | Every finding presented to a human has passed critic review with all 4 structural checks | CRIT-01 through CRIT-04 |
| 9 | Threshold effective-dating works (same metric value, different RAG outcome by date) | POL-02, POL-03 |
| 10 | Committee report includes full evidence citation trace, matching `demo/SAMPLE_EVIDENCE_LEDGER.md` | REP-01 |
| 11 | Full demo runs within the time budget and matches the sample outputs | E2E-06 |
| 12 | Model inventory registry contains all 12 model rows (`data/MODEL_INVENTORY_SCHEMA.md`), even though only 1 carries live monitoring data | Manual review against §2 of that file |

## Sign-off
MVP is accepted for demo when all 12 criteria are checked and the Model Risk Architect and Copilot Studio Solution Architect both confirm E2E-06 against a freshly regenerated dataset (not a previously cached run).
