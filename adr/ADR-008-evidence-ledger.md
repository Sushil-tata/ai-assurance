# ADR-008: Single Evidence Ledger for All Agent Conclusions

**Status:** Accepted
**Date:** 2026-07-13
**Deciders:** Model Risk Architect, Principal Enterprise Data Architect

## Context
Without a common evidence structure, each agent could invent its own way of citing support for a claim, making cross-agent review (the critic loop) and post-hoc audit inconsistent and hard to query.

## Decision
One table, `fact_evidence_item`, is the single structure for every piece of evidence any agent cites, regardless of source (metric value, DQ check, policy clause, prior finding). Every `fact_finding` and `fact_recommendation` row links to evidence exclusively through a bridge table (`fact_finding_evidence_link`), never through free-text description alone.

## Consequences
- Positive: the Assurance/Critic Agent can mechanically verify "does this finding have ≥1 evidence link" as a structural check, not a judgment call.
- Positive: one query pattern reconstructs the full evidence trace for any investigation, used identically in the demo, in `demo/SAMPLE_EVIDENCE_LEDGER.md`, and in real audit.
- Negative: forces every evidence-producing agent (Monitoring, Data Quality, Governance) to write through the same schema rather than a bespoke output — a small design constraint accepted for consistency.

## Alternatives considered
1. **Per-agent evidence tables** — rejected: fragments the audit trail; the critic loop and reporting layer would need bespoke logic per evidence source.
