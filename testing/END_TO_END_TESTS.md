# End-to-End Tests

**Audience:** whoever runs the final pre-demo validation pass.

## E2E-01: Full Scenario 001 trace
**Steps:** Load Scenario 001 synthetic fixtures → run Monitoring Agent scheduled cycle → confirm `INV-2026-0031` created → confirm Investigation Agent produces `FIND-2026-0031-01` with 9 evidence items and 3 contributing factors → confirm Assurance Agent approves (all 4 checks pass) → confirm Reporting Agent posts Teams card → simulate committee "approve" action → confirm `DEC-2026-0014` recorded and `INV-2026-0031` closed → confirm `REM-2026-0031-01` created.
**Pass criteria:** every ID referenced in `scenarios/SCENARIO-001-psi-breach.md` exists in the target Dataverse/Warehouse tables exactly as specified, reachable via the join query in that file's §10.

## E2E-02: Rejection and revision loop
**Steps:** Manually inject a finding with one uncited claim → submit to Assurance Agent → confirm rejection with specific reason → confirm Investigation Agent receives the rejection and revises → confirm resubmission passes.
**Pass criteria:** rejection reason names the specific gap; revised finding differs only in the gap addressed, not wholesale rewritten.

## E2E-03: Two-rejection escalation
**Steps:** Inject a finding that fails critic review twice in a row (same underlying gap not fixed) → confirm the system escalates to human rather than requesting a third revision.
**Pass criteria:** `fact_investigation` shows an escalation event; no third automated critic cycle occurs.

## E2E-04: Applicability violation (Scenario 005) bypasses normal cycle
**Steps:** Inject a B Score fixture with `mob_at_observation < 6` → run Monitoring Agent → confirm `severity=critical` investigation created same-day, independent of the monthly RAG schedule.
**Pass criteria:** investigation exists with correct trigger type; Teams notification fired.

## E2E-05: Closure without decision is structurally impossible
**Steps:** Attempt to directly set `fact_investigation.status = closed` via the API with no `decision_id` supplied.
**Pass criteria:** API returns `MISSING_DECISION` error; no row is updated.

## E2E-06: Full demo dry run
**Steps:** Run the entire `demo/DEMO_SCRIPT.md` sequence against a freshly regenerated synthetic dataset, timed.
**Pass criteria:** completes within the time budget in `DELIVERY_PLAN.md`; every screen/output shown matches `demo/SAMPLE_COMMITTEE_REPORT.md` and `demo/SAMPLE_EVIDENCE_LEDGER.md`.
