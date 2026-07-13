# AI Assurance — Implementation Reference Repository

## What this is

AI Assurance is a Microsoft Copilot Studio–based, multi-agent platform for **continuous model monitoring, investigation, governance and human assurance**. This repository is the full implementation reference: domain model, temporal design, model lifecycle registry, monitoring data mart, investigation/evidence model, agent contracts, Copilot Studio component mapping, policies, test cases and a demo script.

The MVP proves the pattern end to end on **one product (Credit Card), one model (Behaviour / B Score)**, one monitoring cycle and one engineered investigation. The enterprise target extends the same semantic model to all products (Credit Card, Speedy Cash, Speedy Loan) and all model families (A/B/C Score) without redesigning the schema — product and model-family differences are handled through **extension tables**, not through rewriting the core.

## Why this repository exists

Model monitoring tools usually fail in one of two ways: they are dashboards with no reasoning (so a human still has to investigate every breach by hand), or they are LLM chat layers with no deterministic ground truth (so nobody trusts the numbers). AI Assurance is built to avoid both failure modes: **agents reason, tools calculate**. Every number an agent cites was computed by a deterministic pipeline and is traceable to a table and row. See `architecture/AGENT_ARCHITECTURE.md` and `adr/ADR-003-deterministic-metrics.md`.

## How to read this repository

| If you are... | Start here |
|---|---|
| A model risk / governance reviewer | `PROJECT_CHARTER.md` → `policies/` → `data/GOVERNANCE_SCHEMA.md` |
| A data architect | `data/DOMAIN_MODEL.md` → `data/TABLE_CATALOG.md` → `data/ERD.md` → `data/TEMPORAL_MODEL.md` |
| A Copilot Studio builder | `agents/` → `prompts/` → `tools/TOOL_CATALOG.md` → `architecture/SYSTEM_ARCHITECTURE.md` |
| A reviewer deciding whether to fund this | `SCOPE.md` → `MVP_VS_ENTERPRISE.md` → `DELIVERY_PLAN.md` → `RED_TEAM_REVIEW.md` |
| Preparing the SAS Innovate talk | `demo/SAS_INNOVATE_REUSE.md` → `demo/DEMO_SCRIPT.md` → `demo/JUDGE_NARRATIVE.md` |

## Repository map

```
/ai-assurance
├── README.md, PROJECT_CHARTER.md, SCOPE.md, GLOSSARY.md
├── MVP_VS_ENTERPRISE.md, DELIVERY_PLAN.md, RED_TEAM_REVIEW.md
├── architecture/      six cross-cutting architecture views
├── adr/                nine binding architecture decisions
├── data/               domain model, full table catalog, ERD, temporal model,
│                        model inventory schema, monitoring mart, governance
│                        schema, synthetic data spec, data quality rules
├── agents/              seven agent contracts + routing + schemas + evaluation
├── tools/                tool catalog, API/SQL/Power Automate contracts
├── policies/             monitoring policy, thresholds, escalation, remediation
├── scenarios/            six worked breach/failure scenarios
├── prompts/              seven Copilot Studio instruction files (deployable text)
├── schemas/              JSON Schema for agent I/O and tool I/O
├── testing/              test strategy and cases for agents, data, policy, E2E
└── demo/                 demo script, judge narrative, sample outputs
```

## Non-negotiable principles (see `architecture/AGENT_ARCHITECTURE.md` §1 for full rationale)

1. Deterministic calculation is separated from LLM reasoning — agents never compute a metric themselves.
2. Event time and processing time are always recorded separately.
3. No monitoring metric is ever computed on an immature or censored performance population.
4. Every agent conclusion cites a table, row identifier and policy clause — no unsupported claims.
5. Thresholds and escalation rules are policy-as-code, not embedded in agent prompts.
6. No agent takes any action that touches a customer, account or credit decision. Agents investigate and recommend; humans decide.

## Status

MVP scope locked. See `SCOPE.md`. Project name and scope are fixed as **AI Assurance** — no renaming, no alternative framing.

## Ownership

| Area | Owner (role) |
|---|---|
| Overall architecture | Principal Enterprise Data Architect |
| Model risk content | Model Risk Architect |
| Copilot Studio build | Copilot Studio Solution Architect |
| Governance sign-off | Model Risk Committee (MRC) chair |
