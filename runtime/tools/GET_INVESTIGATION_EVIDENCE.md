# Tool: GET_INVESTIGATION_EVIDENCE

## Purpose
Return evidence items for an investigation scope, including supporting and contradictory records.

## Input JSON
```json
{
  "investigation_id": "INV-2026-06-CC-B-01",
  "model_id": "M-CC-B-01",
  "period": "2026-06",
  "hypotheses": [
    "thin_file_campaign_shift",
    "bureau_missingness_issue",
    "policy_population_change"
  ]
}
```

## Output JSON
```json
{
  "investigation_id": "INV-2026-06-CC-B-01",
  "evidence_items": [
    {
      "evidence_id": "EV-2026-06-001",
      "type": "metric_extract",
      "source": "monitoring_mart",
      "summary": "Thin-file share increased from 22% to 31%.",
      "supports": ["thin_file_campaign_shift"],
      "contradicts": [],
      "timestamp": "2026-07-01T10:05:00Z"
    }
  ],
  "generated_at": "2026-07-01T10:20:00Z"
}
```

## Validation
- `investigation_id`, `model_id`, and `period` are required.
- `hypotheses` must contain at least one value.
- Each evidence item must include `evidence_id`, `source`, and `timestamp`.

## Possible Errors
- `INVESTIGATION_NOT_FOUND`
- `NO_EVIDENCE_AVAILABLE`
- `INVALID_HYPOTHESIS_CODE`
- `EVIDENCE_STORE_UNAVAILABLE`

## Example Request
```json
{
  "investigation_id": "INV-2026-06-CC-B-01",
  "model_id": "M-CC-B-01",
  "period": "2026-06",
  "hypotheses": [
    "thin_file_campaign_shift",
    "bureau_missingness_issue"
  ]
}
```

## Example Response
```json
{
  "investigation_id": "INV-2026-06-CC-B-01",
  "evidence_items": [
    {
      "evidence_id": "EV-2026-06-002",
      "type": "data_quality_log",
      "source": "dq_pipeline",
      "summary": "Bureau feature missingness increased by 4.7 percentage points.",
      "supports": ["bureau_missingness_issue"],
      "contradicts": [],
      "timestamp": "2026-07-01T10:10:00Z"
    }
  ],
  "generated_at": "2026-07-01T10:20:00Z"
}
```
