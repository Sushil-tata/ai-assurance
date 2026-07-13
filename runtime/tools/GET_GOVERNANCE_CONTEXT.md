# Tool: GET_GOVERNANCE_CONTEXT

## Purpose
Return current governance policy context, escalation mappings, and approval requirements.

## Input JSON
```json
{
  "model_id": "M-CC-B-01",
  "period": "2026-06",
  "include_thresholds": true,
  "include_escalation": true,
  "include_approval_rules": true
}
```

## Output JSON
```json
{
  "model_id": "M-CC-B-01",
  "period": "2026-06",
  "policy_version": "MONITORING-1.0",
  "threshold_version": "THRESHOLD-1.0",
  "escalation_version": "ESCALATION-1.0",
  "approval_required_for": [
    "recalibration",
    "temporary_use_restriction",
    "retirement_review"
  ],
  "generated_at": "2026-07-01T10:30:00Z"
}
```

## Validation
- `model_id` and `period` are required.
- Version fields must be present in output.
- Approval list must be an array of strings.

## Possible Errors
- `POLICY_CONTEXT_NOT_FOUND`
- `VERSION_REGISTRY_UNAVAILABLE`
- `INVALID_MODEL_SCOPE`

## Example Request
```json
{
  "model_id": "M-CC-B-01",
  "period": "2026-06",
  "include_thresholds": true,
  "include_escalation": true,
  "include_approval_rules": true
}
```

## Example Response
```json
{
  "model_id": "M-CC-B-01",
  "period": "2026-06",
  "policy_version": "MONITORING-1.0",
  "threshold_version": "THRESHOLD-1.0",
  "escalation_version": "ESCALATION-1.0",
  "approval_required_for": [
    "recalibration",
    "redevelopment",
    "temporary_use_restriction"
  ],
  "generated_at": "2026-07-01T10:30:00Z"
}
```
