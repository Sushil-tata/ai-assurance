# Judge / Reviewer Narrative

**Audience:** whoever is evaluating this for SAS Innovate or an internal architecture review, who may not sit through the live demo and instead reads the repository.

## The one-paragraph pitch
Most "AI for model monitoring" demos show an LLM summarizing a dashboard. AI Assurance is built the opposite way: every number is computed deterministically outside the LLM, every agent conclusion cites a specific evidence row, a structurally independent critic agent rejects ungrounded findings before a human ever sees them, and no investigation can close without a named human decision. The interesting engineering is not the chatbot — it's the schema and the governance boundary underneath it.

## What to look at if you only have 10 minutes
1. `scenarios/SCENARIO-001-psi-breach.md` — the complete worked trace, three linked causes, real evidence IDs.
2. `data/TEMPORAL_MODEL.md` §5–7 — the label-maturity/censoring logic, which is the part most monitoring tools get quietly wrong.
3. `agents/ASSURANCE_AGENT.md` — the critic loop, which is what makes the evidence-grounding requirement enforceable rather than aspirational.
4. `adr/ADR-009-mvp-product-model.md` — why the schema handles three structurally different products without becoming a mess of nullable columns.

## What we'd push back on ourselves
See `RED_TEAM_REVIEW.md` — we did not write this repository assuming it's perfect; the red-team section documents real trade-offs (Copilot Studio's branching-logic limits, RAG hallucination risk in the Governance Agent's policy search, the honest limits of automated critic review) rather than pretending the design has no weaknesses.

## Why this generalizes beyond one bank
The core pattern — deterministic metric layer, evidence ledger, mandatory critic review, human-gated closure — is not specific to credit risk. The same architecture applies to any regulated monitoring domain where "the AI said so" is not an acceptable answer: fraud model monitoring, AML alerting, insurance underwriting model governance. Credit Card Behaviour Score is the concrete proof point, not the ceiling of the idea.
