# Approval Card Specification

## Title
AI Assurance Governance Decision Request

## Fields
- Decision ID
- Model ID
- Period
- Severity
- Recommended Action
- Alternatives Considered
- Evidence IDs
- Rationale
- Action Owner
- Due Date

## Buttons
- Approve
- Approve with Modification
- Request Further Analysis
- Reject
- Defer

## Actions
- `Approve`: set decision status to approved.
- `Approve with Modification`: require modification text and rationale.
- `Request Further Analysis`: require analysis scope and due date.
- `Reject`: require rejection rationale.
- `Defer`: require defer reason and review date.

## Validation Rules
- All actions require actor identity.
- Override actions require mandatory rationale text.
- Decision cannot be submitted without at least one evidence ID.
