# ADR-002: Microsoft Copilot Studio as the Agent Runtime

**Status:** Accepted
**Date:** 2026-07-13
**Deciders:** Copilot Studio Solution Architect, Principal Enterprise Data Architect

## Context
The multi-agent system needs an orchestration runtime, a human interface (Teams), low-code tool/action wiring, and enterprise identity integration, buildable by a small team on a Microsoft-standard stack.

## Decision
Use **Copilot Studio connected agents** for all seven agents, Power Automate for scheduling/notification, Dataverse for the governance layer, Teams as the human interface. Reject a custom-coded agent framework (e.g., raw Azure OpenAI + LangChain-style orchestration).

## Consequences
- Positive: native Teams integration, native Dataverse actions, native Entra ID identity — least-privilege enforcement (Principle 1.8) is a configuration exercise, not custom code.
- Positive: connected-agent hand-off model maps directly onto the Orchestrator → specialist topology in `architecture/AGENT_ARCHITECTURE.md`.
- Negative: Copilot Studio's action/topic model has less flexibility than raw code for complex branching logic — mitigated by pushing all non-trivial logic into deterministic tools (Azure Functions/SQL), keeping agent-side logic to routing and reasoning only (this reinforces, not conflicts with, Principle 1.1).
- Negative: harder to unit-test agent instructions in isolation compared to code — mitigated by `testing/AGENT_TEST_CASES.md` using scripted conversation transcripts as the test unit.

## Alternatives considered
1. **Custom orchestration (Azure OpenAI + code)** — rejected for MVP: higher build cost, loses native Teams/Dataverse/Entra integration, and the brief specifies Copilot Studio explicitly.
2. **Single monolithic Copilot Studio agent (no connected agents)** — rejected: violates least-privilege and auditability principles (1.8, 1.9) — a single agent with every tool wired in cannot express per-agent data-access boundaries.
