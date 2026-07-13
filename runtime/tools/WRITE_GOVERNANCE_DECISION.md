# Tool: WRITE_GOVERNANCE_DECISION

## Purpose
Persist a governance decision draft or final human decision record with rationale and evidence references.

## Input JSON
```json
{
  "decision_id": "DEC-2026-06-CC-B-01",
  "investigation_id": "INV-2026-06-CC-B-01",
  "model_id": "M-CC-B-01",
  "decision_type": "approve_with_modification",
  "rationale": "Proceed with enhanced monitoring and data remediation before recalibration.",
  "evidence_ids": ["EV-2026-06-001", "EV-2026-06-002", "EV-2026-06-004"],
  "action_owner": "Model Owner",
  "due_date": "2026-08-15",
  "approval_status": "pending_human_approval"
}
```

## Output JSON
```json
{
  "decision_id": "DEC-2026-06-CC-B-01",
  "status": "written",
  "record_version": 1,
  "written_at": "2026-07-02T08:00:00Z"
}
```

## Validation
- `decision_id`, `model_id`, `decision_type`, `rationale`, and `approval_status` are required.
- `decision_type` must be one of approve, approve_with_modification, request_further_analysis, reject, defer.
- `evidence_ids` must contain at least one ID.

## Possible Errors
- `INVALID_DECISION_TYPE`
- `MISSING_RATIONALE`
- `EVIDENCE_REFERENCE_NOT_FOUND`
- `DECISION_STORE_UNAVAILABLE`

## Example Request
```json
{
  "decision_id": "DEC-2026-06-CC-B-01",
  "investigation_id": "INV-2026-06-CC-B-01",
  "model_id": "M-CC-B-01",
  "decision_type": "request_further_analysis",
  "rationale": "Need additional counter-evidence on policy cohort effect.",
  "evidence_ids": ["EV-2026-06-003", "EV-2026-06-007"],
  "action_owner": "Model Risk",
  "due_date": "2026-07-20",
  "approval_status": "pending_human_approval"
}
```

## Example Response
```json
{
  "decision_id": "DEC-2026-06-CC-B-01",
  "status": "written",
  "record_version": 2,
  "written_at": "2026-07-02T08:10:00Z"
}
```
