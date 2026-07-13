# Assurance Flow

## Input
- Monitoring summary.
- Investigation result.
- Evidence package.

## Routing
1. AI Assurance Master routes to Assurance Specialist.
2. Assurance Specialist checks evidence completeness and policy references.
3. If pass or pass with caveats, route to Governance Specialist.
4. Governance Specialist calls GET_GOVERNANCE_CONTEXT.
5. Governance Specialist drafts recommendation and calls WRITE_GOVERNANCE_DECISION.

## Output
- Assurance verdict.
- Governance recommendation.
- Decision draft record.

## Exceptions
- Evidence gaps: return block with missing items list.
- Policy version mismatch: escalate to governance owner.
- Decision write failure: retry once, then hold for manual entry.
