# Project Charter — AI Assurance

**Audience:** sponsors, Model Risk Committee, enterprise architecture review board.
**Status:** locked. Project name, scope framing and product are fixed.

## 1. Problem statement

Credit risk models at the bank are monitored today through monthly Excel/SAS packs that a model risk analyst assembles by hand, interprets, and writes up for committee. This has three structural weaknesses:

1. **Latency.** A PSI breach in June is often not investigated until the July or August pack, because investigation is manual and analyst time is scarce.
2. **Inconsistency.** Two analysts investigating the same breach can reach different root causes because there is no shared evidence ledger — conclusions live in slide decks, not in queryable records.
3. **No systematic linkage between monitoring and governance.** A metric turning Amber does not automatically open an investigation, does not automatically check model applicability (e.g., MOB eligibility), and does not automatically produce a citable audit trail from breach to committee decision.

## 2. What AI Assurance is

A Microsoft Copilot Studio multi-agent system that:
- runs deterministic monitoring calculations on a governed data mart (never on live production data directly);
- opens a structured investigation automatically when a metric breaches policy threshold;
- gathers evidence (metric values, feature drift, data-quality signals, policy clauses) into an auditable evidence ledger;
- proposes findings, root causes and recommendations, reviewed by a critic agent before any human sees them;
- routes every material conclusion through a human approval / committee decision step before it is considered final;
- produces a committee-ready report with full evidence lineage.

## 3. What AI Assurance is not

- Not an autonomous decisioning system. No agent approves, declines, prices, or changes a customer's credit line or collections treatment.
- Not a model development platform. Model training happens outside this system; AI Assurance monitors what is already deployed.
- Not a general-purpose chatbot over the data warehouse. Agents have named, bounded tool access — see `agents/*.md`.

## 4. MVP commitment

Full end-to-end demonstration of:
- Credit Card Behaviour Score (B Score), monthly scoring, MOB ≥ 6 eligibility;
- forward performance monitoring at 3, 6 and 12 months;
- one engineered PSI-breach investigation carried from trigger to committee decision;
- seven Copilot Studio agents, four to six tools, one approval flow, one committee report;
- synthetic but internally consistent data (see `data/SYNTHETIC_DATA_SPEC.md`).

See `SCOPE.md` and `MVP_VS_ENTERPRISE.md` for the exact in/out boundary.

## 5. Enterprise target

The same semantic model extended to Speedy Cash and Speedy Loan, all three model families (A/B/C), full model lifecycle management, the full monitoring metric set, and production-grade controls (Purview lineage, Entra-based least privilege, Application Insights tracing). The MVP schema is designed so this extension requires **adding rows and extension tables, not redesigning the core** — this is the central architectural bet of the project and is defended in `adr/ADR-009-mvp-product-model.md`.

## 6. Success criteria

| Criterion | Evidence |
|---|---|
| A monitoring breach is detected without human polling | Monitoring Agent scheduled run + Orchestrator routing log |
| The breach is investigated with evidence, not opinion | `demo/SAMPLE_EVIDENCE_LEDGER.md` |
| A human decision is required before closure | Committee decision record with named approver |
| The whole trace is queryable after the fact | Evidence ledger joins investigation → finding → recommendation → decision |
| The design generalizes beyond Credit Card B Score without schema rewrite | `MVP_VS_ENTERPRISE.md` gap analysis |

## 7. Governance and accountability

Model Risk Committee retains decision authority. AI Assurance agents are classified as **decision-support tools** under the Agent Risk Policy (`policies/AGENT_RISK_POLICY.md`), not as automated decision systems. Every agent-produced recommendation carries a named human accountable owner before it can close an investigation.

## 8. Sponsors and reviewers

| Role | Responsibility |
|---|---|
| Chief Model Risk Officer (sponsor) | Governance sign-off, committee chair |
| Head of Enterprise Data Architecture | Domain model and lineage sign-off |
| Copilot Studio Solution Architect | Build feasibility, agent boundary sign-off |
| Model Risk Committee | Final approval authority for MVP demo findings |
