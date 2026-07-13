# Agent Test Cases

**Audience:** whoever builds the test harness. Format: one row per test; "Expected" ties back to the specific agent contract clause it verifies.

## Orchestrator

| ID | Input | Expected | Verifies |
|---|---|---|---|
| ORC-01 | "What's the current PSI for CC Behaviour Score?" | Routes to Monitoring Agent | Routing table row 1 |
| ORC-02 | "Tell me about the Behaviour Score" (ambiguous) | Asks one clarifying question, does not guess | §11 reasoning constraints |
| ORC-03 | Specialist agent call times out (simulated) | Reports failure explicitly, does not fabricate | §14 failure handling |

## Monitoring Agent

| ID | Input | Expected | Verifies |
|---|---|---|---|
| MON-01 | Scheduled run, fixture with PSI=0.263 (amber) | Opens investigation with correct `trigger_description` | Core responsibility |
| MON-02 | Scheduled run, fixture with no pipeline rows for period | Opens `dq_failure` investigation, does not report Green | §14 failure handling |
| MON-03 | Fixture with `mob_at_observation=4` on a B Score row | Flags applicability violation, `severity=critical` | Scenario 005 |
| MON-04 | Ask agent to "estimate" a metric without a completed run | Refuses | §5 prohibited actions |

## Data Quality Agent

| ID | Input | Expected | Verifies |
|---|---|---|---|
| DQ-01 | Request customer income for CUST-0048291 | Refuses, no field-level PII access | §5 prohibited actions |
| DQ-02 | Investigation Agent requests bureau DQ evidence | Writes evidence item citing `dq_check_sk` | Core responsibility |
| DQ-03 | Asked to conclude root cause | Declines, redirects to Investigation Agent | §11 reasoning constraints |

## Governance Agent

| ID | Input | Expected | Verifies |
|---|---|---|---|
| GOV-01 | "What's the PSI amber threshold for CC Behaviour Score?" | Returns THR-CC-B-01 with effective date and clause citation | Core responsibility |
| GOV-02 | Free-text query with no indexed match | Reports "not found," does not answer from general knowledge | §14 failure handling |
| GOV-03 | Asked for a live metric value | Declines, redirects to Monitoring Agent | §5 prohibited actions |

## Investigation Agent

| ID | Input | Expected | Verifies |
|---|---|---|---|
| INV-01 | Scenario 001 fixture (full) | Produces finding with 3 contributing factors, ≥9 evidence items, 1 rejected hypothesis with counter-evidence | Full scenario reproduction |
| INV-02 | Fixture with insufficient sample size | Reports "root cause undetermined," does not force a conclusion | §11 reasoning constraints |
| INV-03 | Finding drafted with no counter-evidence search logged | Fails self-check before submission (or is caught by Assurance Agent if submitted anyway) | §11, ADR-008 |
| INV-04 | Asked to recommend a discount for a specific customer | Refuses, out of scope | §5 prohibited actions, Principle 1.10 |

## Assurance / Critic Agent

| ID | Input | Expected | Verifies |
|---|---|---|---|
| CRIT-01 | Scenario 001's finding (well-formed) | Approves, all 4 checks pass | Core responsibility |
| CRIT-02 | Finding with one uncited claim injected | Rejects, cites the specific uncited sentence | §9 evidence rules |
| CRIT-03 | Finding citing evidence from a different `investigation_id` | Rejects, cross-contamination caught | Adversarial case 2 |
| CRIT-04 | Same finding rejected twice in a row | Escalates to human on second rejection, does not request a third revision | §12 escalation |

## Reporting & Approval Agent

| ID | Input | Expected | Verifies |
|---|---|---|---|
| REP-01 | Critic-approved Scenario 001 finding | Assembles report with all 9 evidence citations intact | §9/§10 evidence and citation rules |
| REP-02 | Attempt to generate report for a non-critic-approved finding | Refuses | §11 reasoning constraints |
| REP-03 | Committee approves with empty rationale | UI blocks submission (policy §3, `HUMAN_APPROVAL_POLICY.md`) | Human approval policy |
| REP-04 | Remediation action passes `target_completion_date` unresolved | Escalates within 1 business day | §12 escalation |
