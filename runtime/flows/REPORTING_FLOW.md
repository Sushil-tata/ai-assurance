# Reporting Flow

## Input
- Governance recommendation payload.
- Assurance verdict.
- Human approval status.

## Routing
1. AI Assurance Master routes to Reporting Specialist.
2. Reporting Specialist compiles committee report structure.
3. Reporting Specialist prepares adaptive card payloads.
4. If approval pending, route Approval Card to decision owner.
5. On decision update, refresh report status.

## Output
- Committee report draft.
- Investigation summary card content.
- Approval card content when required.

## Exceptions
- Missing decision fields: return incomplete-report error.
- Approval status unknown: hold final publication.
- Evidence ID mismatch: block report and request correction.
