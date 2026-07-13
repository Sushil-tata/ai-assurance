# ADR-005: Mandatory Human Approval Before Closure

**Status:** Accepted
**Date:** 2026-07-13
**Deciders:** Model Risk Architect (chair authority), Principal Enterprise Data Architect

## Context
An automated monitoring/investigation system could, technically, close its own investigations once the Assurance/Critic Agent approves a finding. This would be faster but removes human accountability from a governance process that regulators and internal audit expect to be human-owned.

## Decision
`fact_investigation.status` can only reach `closed` through a non-null `decision_id` FK to `fact_committee_decision` or `fact_human_approval`, naming a real Entra identity. No agent, including the Assurance/Critic Agent, can set this field directly.

## Consequences
- Positive: AI Assurance remains structurally a decision-support system (Principle 1.7, 1.10), not an autonomous governance system — a materially different (and much lighter) regulatory posture.
- Positive: gives the Model Risk Committee a natural, unavoidable checkpoint to catch any residual agent error the critic loop missed.
- Negative: introduces latency — an investigation cannot fully close outside a human's calendar. Accepted; this system optimizes time-to-evidence, not time-to-closure.

## Alternatives considered
1. **Auto-close low-severity investigations** — rejected for MVP: even a "low severity" auto-close sets a precedent that erodes the human-accountability principle; can be revisited as a narrowly scoped enterprise increment with its own ADR if ever proposed.
