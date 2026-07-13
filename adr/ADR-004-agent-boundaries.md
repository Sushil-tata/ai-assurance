# ADR-004: Hard Per-Agent Data and Tool Boundaries

**Status:** Accepted
**Date:** 2026-07-13
**Deciders:** Copilot Studio Solution Architect, Model Risk Architect

## Context
A multi-agent system where every agent can call every tool and read every table is easier to build initially but impossible to audit or secure, and makes it unclear which agent is "responsible" for a given class of error.

## Decision
Each of the seven agents has an explicit, disjoint-where-possible tool and data-domain allow-list, documented in its contract (`agents/*.md`) and enforced at the Copilot Studio connector/topic configuration layer, not only in the system prompt. Overlap is permitted only where a genuine cross-check is required (e.g., both Investigation and Assurance agents can read the evidence ledger, by design, for the critic loop).

## Consequences
- Positive: `architecture/SECURITY_ARCHITECTURE.md`'s access matrix is enforceable, not aspirational.
- Positive: a failure or hallucination is traceable to a specific agent's specific tool call in `fact_tool_execution_log`.
- Negative: some duplication of small helper actions across agents (e.g., both Monitoring and Investigation agents need to read `fact_monitoring_metric`) — accepted, this is a feature of least-privilege design, not a code-reuse concern.

## Alternatives considered
1. **One shared toolset for all agents** — rejected: violates Principle 1.8, makes root-cause attribution of agent errors impossible.
