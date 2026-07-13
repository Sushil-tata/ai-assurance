# ADR-007: Policy-as-Code for Thresholds and Eligibility Rules

**Status:** Accepted
**Date:** 2026-07-13
**Deciders:** Model Risk Architect, Governance Agent design owner

## Context
Monitoring thresholds (RAG bands), eligibility rules (MOB ≥ 6), and escalation rules change over time and must themselves be governed — a threshold change is itself a risk decision.

## Decision
All thresholds, RAG bands, eligibility rules and escalation rules are structured rows in `dim_governance_threshold`, versioned with `effective_start_date` / `effective_end_date`, each linked to the `dim_policy_clause` that authorizes it and the human approver who set it. No threshold value appears in any agent prompt, tool code constant, or Power Automate flow condition — every threshold check is a runtime lookup.

## Consequences
- Positive: a threshold change is auditable exactly like a model change — who, when, why, under what policy clause.
- Positive: the Governance Agent can answer "what was the PSI threshold in March?" from history, not just "what is it now."
- Negative: every threshold-consuming tool must perform a lookup rather than a constant comparison — negligible performance cost, worth the governance benefit.

## Alternatives considered
1. **Thresholds embedded in agent instructions** — rejected: violates Principle 1.5, makes threshold changes indistinguishable from prompt engineering in the change log.
