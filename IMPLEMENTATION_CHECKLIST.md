# Final Implementation Checklist

**Audience:** builder and sponsor, as a single-page go/no-go reference. Every row links to the file that proves it.

## Design and governance
- [ ] 11 design principles reviewed and accepted — `architecture/AGENT_ARCHITECTURE.md` §1
- [ ] 9 ADRs reviewed and accepted — `adr/`
- [ ] Agent Risk Policy classification (decision-support, not autonomous) accepted — `policies/AGENT_RISK_POLICY.md`
- [ ] Human Approval Policy accepted — `policies/HUMAN_APPROVAL_POLICY.md`

## Data
- [ ] All 45 tables created in Fabric Warehouse / Dataverse — `data/TABLE_CATALOG.md`
- [ ] ERD reviewed against actual DDL — `data/ERD.md`
- [ ] Temporal rules (maturity, censoring, leakage) implemented and tested — `data/TEMPORAL_MODEL.md`, `testing/DATA_TEST_CASES.md`
- [ ] Model inventory populated with all 12 rows — `data/MODEL_INVENTORY_SCHEMA.md`
- [ ] Monitoring mart metrics implemented for CC B Score — `data/MONITORING_MART.md`
- [ ] Data quality rules implemented — `data/DATA_QUALITY_RULES.md`
- [ ] Synthetic dataset generated and passes fixture-integrity checks — `data/SYNTHETIC_DATA_SPEC.md`

## Agents
- [ ] All 7 agents built in Copilot Studio — `agents/`, `prompts/`
- [ ] Per-agent data/tool boundaries enforced at connector level, not just documented — `architecture/SECURITY_ARCHITECTURE.md`
- [ ] Agent routing wired — `agents/AGENT_ROUTING.md`
- [ ] Critic loop wired and circuit-breaker tested — `agents/ASSURANCE_AGENT.md`, E2E-03

## Tools and integration
- [ ] 6 MVP tools built — `tools/TOOL_CATALOG.md`
- [ ] Power Automate flows built — `tools/POWER_AUTOMATE_FLOWS.md`
- [ ] Error handling rules implemented — `tools/ERROR_HANDLING.md`
- [ ] Azure AI Search policy index built and tuned — `architecture/SYSTEM_ARCHITECTURE.md` §1

## Governance workflow
- [ ] Investigation state machine enforced (no closure without decision) — `data/GOVERNANCE_SCHEMA.md` §3, E2E-05
- [ ] Escalation matrix rules implemented — `policies/ESCALATION_MATRIX.md`
- [ ] Remediation tracking implemented — `policies/REMEDIATION_PLAYBOOK.md`
- [ ] Teams approval card working end to end — Step 10, `DELIVERY_PLAN.md`

## Testing
- [ ] All data test cases pass — `testing/DATA_TEST_CASES.md`
- [ ] All agent test cases pass — `testing/AGENT_TEST_CASES.md`
- [ ] All policy test cases pass — `testing/POLICY_TEST_CASES.md`
- [ ] All end-to-end tests pass — `testing/END_TO_END_TESTS.md`
- [ ] Full acceptance criteria checked — `testing/ACCEPTANCE_CRITERIA.md`

## Demo readiness
- [ ] Scenario 001 reproduces cleanly from a fresh environment — E2E-01, E2E-06
- [ ] Demo script rehearsed with fallback recording ready — `demo/DEMO_SCRIPT.md`
- [ ] Sample committee report and evidence ledger match live output — `demo/SAMPLE_COMMITTEE_REPORT.md`, `demo/SAMPLE_EVIDENCE_LEDGER.md`

## Honest self-review
- [ ] Red-team review read and open items triaged — `RED_TEAM_REVIEW.md`
- [ ] MVP/Enterprise boundary communicated accurately to sponsors, not overstated — `MVP_VS_ENTERPRISE.md`
- [ ] Known gaps (application-score denormalization reconciliation, source-row-identifier runtime validation, Orchestrator hop-count limit) logged as follow-ups, not silently dropped — `RED_TEAM_REVIEW.md` §3, §4, §12

## Sign-off
| Role | Name | Date | Signature |
|---|---|---|---|
| Model Risk Architect | | | |
| Copilot Studio Solution Architect | | | |
| Chief Model Risk Officer (sponsor) | | | |
