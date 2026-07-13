# SAS Innovate Reuse Notes

**Audience:** presenter adapting this repository for a SAS Innovate talk or paper.

## 1. What transfers directly
- The architectural pattern (deterministic metric layer + evidence-grounded agentic reasoning + mandatory critic review + human-gated closure) is platform-agnostic. It is illustrated here on Microsoft Copilot Studio, but the talking points in `architecture/AGENT_ARCHITECTURE.md` §1 (the eleven design principles) apply equally to a SAS Viya / SAS Model Manager-centric implementation.
- `data/TEMPORAL_MODEL.md` and `data/MONITORING_MART.md` are directly reusable as a model-monitoring reference architecture talk on their own, independent of the agent layer — many SAS Innovate audiences are model risk practitioners first, agentic-AI-curious second.
- `scenarios/SCENARIO-001-psi-breach.md` is a strong standalone case study: "here is a monitoring breach with three simultaneous, entangled causes, and here is a fully traceable resolution" resonates with a model risk audience regardless of which vendor stack they use.

## 2. Suggested talk structure (45 minutes)
1. The problem with monitoring-as-dashboard (5 min) — `PROJECT_CHARTER.md` §1.
2. Why one model can't do everything (5 min) — the A/B/C Score distinction, `GLOSSARY.md` model-family section.
3. The temporal traps that break monitoring silently (10 min) — `data/TEMPORAL_MODEL.md` §5, immature/censored/leakage.
4. Deterministic-plus-agentic architecture (10 min) — `architecture/AGENT_ARCHITECTURE.md` topology diagram.
5. Walk the evidence trace live (10 min) — Scenario 001, from breach to committee decision.
6. Governance and human accountability (5 min) — `architecture/HUMAN_OVERSIGHT.md`, why this isn't autonomous decisioning.

## 3. What to reframe for a SAS-centric audience
- Fabric Warehouse / Dataverse → generically "a governed monitoring mart + a governed case-management layer"; the specific vendor is incidental to the pattern.
- Copilot Studio agents → generically "bounded, auditable reasoning agents with least-privilege tool access"; the same boundary discipline applies whether built on Copilot Studio, a SAS Intelligent Decisioning-adjacent stack, or a custom framework.
- Keep the schema and temporal model slides vendor-neutral — they are the most durable, most broadly applicable content in this repository.

## 4. What NOT to overclaim
Do not present this as a production-proven system — it is an MVP reference architecture with synthetic data. Be explicit that the depth is in the design (schema, temporal logic, governance boundary), not in operational track record. This is more credible to a model risk audience than overstating maturity.
