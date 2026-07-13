# Investigation Flow

## Input
- Investigation request payload.
- Triggering monitoring summary.

## Routing
1. AI Assurance Master routes to Investigation Specialist.
2. Investigation Specialist calls GET_INVESTIGATION_EVIDENCE.
3. Hypothesis and counter-evidence testing is performed.
4. If evidence quality is questionable, route to Assurance Specialist.
5. Route final result to Governance Specialist.

## Output
- Investigation result payload with primary cause, contributing factors, unrelated signals.
- Confidence level.
- Evidence references.

## Exceptions
- No evidence returned: escalate for further analysis.
- Contradictory evidence unresolved: mark confidence Low and escalate.
- Schema validation failure: reject payload and re-request evidence.
