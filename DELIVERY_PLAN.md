# Delivery Plan (Part 12)

**Audience:** a single builder executing this end to end.
**Total estimated effort:** 7–9 working weeks for one experienced person spanning data engineering, Copilot Studio, and model risk domain knowledge — realistically closer to 9–11 weeks if any of those three skill areas is not already strong (see `RED_TEAM_REVIEW.md` §"unrealistic MVP scope").

## Step 1 — Repository
**Deliverable:** this repository, scaffolded and committed to source control.
**Prerequisite:** none.
**Effort:** 0.5 day.
**Tools:** Git, markdown editor.
**Definition of done:** folder structure matches `README.md`'s map; every file referenced elsewhere exists (even if a stub at this stage).
**Risk:** scope creep into "perfect documentation before any code" — timebox strictly.
**Fallback:** ship with fewer scenario files (2 instead of 6) if time-constrained; Scenario 001 is non-negotiable, others are regression-test nice-to-haves.

## Step 2 — ADRs
**Deliverable:** `adr/ADR-001` through `ADR-009`.
**Prerequisite:** Step 1.
**Effort:** 1 day.
**Tools:** none beyond markdown.
**Definition of done:** every ADR has a Decision and Consequences section; no ADR is a placeholder.
**Risk:** ADRs written but not actually followed during build — mitigate by referencing the specific ADR in code/config comments during later steps.
**Fallback:** none — this is cheap and high-value, do not cut it.

## Step 3 — Data contracts
**Deliverable:** `data/TABLE_CATALOG.md`, `data/DOMAIN_MODEL.md`, `data/ERD.md`, `data/TEMPORAL_MODEL.md` finalized; DDL scripts generated from the catalog for Fabric Warehouse and Dataverse.
**Prerequisite:** Step 2 (especially ADR-006, ADR-009).
**Effort:** 5 days (schema is already fully specified in this repository; this step is implementation, not (re)design).
**Tools:** Fabric Warehouse, Dataverse solution explorer, SQL DDL scripts.
**Definition of done:** all 45 tables exist in their target platform with correct keys; `testing/DATA_TEST_CASES.md` referential-integrity checks pass on an empty (schema-only) load.
**Risk:** underestimating Dataverse table/relationship configuration time (it is slower to configure via UI than raw SQL DDL).
**Fallback:** for MVP, the Dataverse governance-layer tables (Domain 21–32) can be built first and fastest since they're the ones agents interact with live; Fabric Warehouse tables for SPC/SPL extensions can be stubbed (schema only, no data) without blocking the demo.

## Step 4 — Synthetic data
**Deliverable:** full synthetic dataset per `data/SYNTHETIC_DATA_SPEC.md`, loaded into Fabric Lakehouse/Warehouse.
**Prerequisite:** Step 3.
**Effort:** 6 days (generator scripting + tuning realism + encoding the three Scenario 001 causes correctly).
**Tools:** Python (pandas/numpy or a synthetic-data library), Fabric notebooks.
**Definition of done:** `testing/DATA_TEST_CASES.md` full suite passes, including DATA-14/15/16 (Scenario 001 fixture integrity).
**Risk:** synthetic data that's statistically incoherent (score doesn't predict outcome) undermines demo credibility — budget explicit tuning time, don't treat this as "just generate random numbers."
**Fallback:** if time-constrained, generate only 12 months of history instead of 18 — reduces realism of vintage/MOB trend visuals but doesn't break Scenario 001.

## Step 5 — Deterministic metric layer
**Deliverable:** Azure Function / SQL pipeline computing all metrics in `data/MONITORING_MART.md` for CC B Score, writing to `fact_monitoring_metric` / `fact_segment_monitoring_metric`.
**Prerequisite:** Step 4.
**Effort:** 6 days.
**Tools:** Azure Functions, SQL, Python (for PSI/Gini/KS/Brier calculation logic).
**Definition of done:** `testing/POLICY_TEST_CASES.md` full suite passes (especially the effective-dated threshold pair, POL-02/POL-03); running against the Scenario 001 fixture data produces PSI ≈ 0.263 for June.
**Risk:** this is the step most likely to reveal that the synthetic data (Step 4) doesn't actually produce the intended metric value — budget iteration time between Steps 4 and 5, don't treat them as strictly sequential.
**Fallback:** none — this is core to the whole demo; do not compromise on it.

## Step 6 — Tools
**Deliverable:** the 6 MVP tools in `tools/TOOL_CATALOG.md`, wired as Copilot Studio actions / Power Automate flows / Azure AI Search index.
**Prerequisite:** Step 5 (tools read from the metric layer) and Step 3 (Dataverse governance tables).
**Effort:** 4 days.
**Tools:** Copilot Studio, Power Automate, Azure AI Search.
**Definition of done:** each tool independently callable and returns schema-conformant responses per `tools/API_CONTRACTS.md`.
**Risk:** Azure AI Search index quality (relevance of policy clause retrieval) — budget time to tune chunking/embedding of the policy documents.
**Fallback:** if Azure AI Search tuning is slow, MVP can launch with a smaller, hand-curated set of indexed clauses (the ones Scenario 001 actually needs) rather than the full policy library.

## Step 7 — Specialist agents
**Deliverable:** Monitoring, Data Quality, Governance, Investigation, Assurance/Critic, Reporting & Approval agents built in Copilot Studio using `prompts/*_INSTRUCTIONS.md`.
**Prerequisite:** Step 6.
**Effort:** 8 days (Investigation and Assurance agents are the most complex, budget disproportionate time there).
**Tools:** Copilot Studio.
**Definition of done:** `testing/AGENT_TEST_CASES.md` per-agent tests pass individually (before wiring the Orchestrator).
**Risk:** Copilot Studio's instruction-following on multi-step reasoning (hypothesis generation, counter-evidence search) may need more explicit step-by-step scaffolding than the prompt file alone provides — budget prompt-iteration time.
**Fallback:** if the Investigation Agent's autonomous hypothesis generation proves unreliable, fall back to a more scripted flow for the MVP demo specifically (a semi-guided sequence of tool calls) while keeping the fully autonomous version as the target for iteration post-demo.

## Step 8 — Orchestrator
**Deliverable:** Orchestrator agent with connected-agent routing per `agents/AGENT_ROUTING.md`.
**Prerequisite:** Step 7.
**Effort:** 2 days.
**Tools:** Copilot Studio.
**Definition of done:** `testing/AGENT_TEST_CASES.md` ORC-01 through ORC-03 pass.
**Risk:** low.
**Fallback:** none needed.

## Step 9 — Critic loop
**Deliverable:** Assurance/Critic Agent wired to trigger automatically on finding submission (Flow-03/04).
**Prerequisite:** Step 7, 8.
**Effort:** 2 days (agent itself built in Step 7; this step is the event-driven wiring).
**Tools:** Power Automate, Dataverse triggers.
**Definition of done:** E2E-02 and E2E-03 (rejection/revision, two-rejection escalation) pass.
**Risk:** Dataverse trigger latency — verify triggers fire promptly enough for a live demo (seconds, not minutes).
**Fallback:** if trigger latency is a problem for the live demo, a manual "check for pending review" button can substitute without changing the underlying architecture.

## Step 10 — Approval
**Deliverable:** Reporting & Approval Agent, Teams adaptive card, decision recording.
**Prerequisite:** Step 9.
**Effort:** 3 days.
**Tools:** Copilot Studio, Power Automate, Teams.
**Definition of done:** E2E-05 (closure without decision is impossible) and REP-03 (empty rationale blocked) pass.
**Risk:** Adaptive Card action-to-Dataverse-write latency in Teams — test early, this is a visible demo moment.
**Fallback:** none — this is the human-accountability payoff moment of the whole demo, do not compromise.

## Step 11 — Reporting
**Deliverable:** committee report assembly matching `demo/SAMPLE_COMMITTEE_REPORT.md`, Power BI dashboard for the monitoring pack.
**Prerequisite:** Step 10.
**Effort:** 3 days.
**Tools:** Power Automate, Power BI.
**Definition of done:** generated report for Scenario 001 matches the sample file's structure and evidence completeness.
**Risk:** low.
**Fallback:** a well-formatted markdown/Word export can substitute for a full Power BI embed if time-constrained — the evidence completeness matters more than the visual polish.

## Step 12 — Testing
**Deliverable:** full test suite execution per `testing/TEST_STRATEGY.md`, all `ACCEPTANCE_CRITERIA.md` items checked.
**Prerequisite:** Steps 1–11.
**Effort:** 4 days.
**Tools:** whatever test harness was built alongside each step (recommend building tests incrementally per step, not entirely at the end, despite this plan listing it as a discrete step for clarity).
**Definition of done:** `testing/ACCEPTANCE_CRITERIA.md` fully checked.
**Risk:** discovering a schema or agent-boundary issue late — mitigated by running `testing/DATA_TEST_CASES.md` and `testing/AGENT_TEST_CASES.md` incrementally from Step 3/7 onward, not deferring all testing to Step 12.
**Fallback:** none — do not demo an untested system.

## Step 13 — Demo
**Deliverable:** rehearsed run of `demo/DEMO_SCRIPT.md` against a freshly regenerated dataset.
**Prerequisite:** Step 12.
**Effort:** 2 days (rehearsal + fallback recording per the script's fallback plan).
**Tools:** Teams, Power BI, screen recording software (fallback).
**Definition of done:** E2E-06 passes; a recorded fallback exists for steps 3–6 of the demo script in case live agent calls are unreliable in front of an audience.
**Risk:** live-demo unreliability under audience pressure (network, Copilot Studio latency).
**Fallback:** the demo script's own fallback plan — pre-recorded steps 3–6, live for the rest.

## Total effort summary
| Steps | Days |
|---|---|
| 1–2 (foundation) | 1.5 |
| 3–5 (data + metrics) | 17 |
| 6–11 (agents + workflow) | 22 |
| 12–13 (test + demo) | 6 |
| **Total** | **~46.5 working days (≈ 9.5 weeks)** |
