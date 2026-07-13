# Tool: GET_MODEL_MONITORING_SUMMARY

## Purpose
Return deterministic monitoring metrics and status for a specific model, period, and segment scope.

## Input JSON
```json
{
  "model_id": "M-CC-B-01",
  "period": "2026-06",
  "product": "Credit Card",
  "include_segments": ["thin_file", "policy_cohort", "channel"],
  "include_data_quality": true
}
```

## Output JSON
```json
{
  "model_id": "M-CC-B-01",
  "period": "2026-06",
  "status": "Amber",
  "metrics": {
    "psi": 0.18,
    "gini_change_pp": -2.5,
    "oe_ratio": 1.14,
    "applicability_violation_rate": 0.0
  },
  "signals": [
    {
      "code": "THIN_FILE_MIX_SHIFT",
      "severity": "Amber",
      "evidence_id": "EV-2026-06-001"
    }
  ],
  "generated_at": "2026-07-01T10:00:00Z"
}
```

## Validation
- `model_id`, `period`, and `product` are required.
- `period` must be `YYYY-MM`.
- `status` must be Green, Amber, or Red.
- Metric fields must be numeric.

## Possible Errors
- `MODEL_NOT_FOUND`
- `PERIOD_NOT_AVAILABLE`
- `INVALID_SCOPE`
- `UPSTREAM_METRIC_SERVICE_UNAVAILABLE`

## Example Request
```json
{
  "model_id": "M-CC-B-01",
  "period": "2026-06",
  "product": "Credit Card",
  "include_segments": ["thin_file", "policy_cohort"],
  "include_data_quality": true
}
```

## Example Response
```json
{
  "model_id": "M-CC-B-01",
  "period": "2026-06",
  "status": "Amber",
  "metrics": {
    "psi": 0.18,
    "gini_change_pp": -2.5,
    "oe_ratio": 1.14,
    "applicability_violation_rate": 0.0
  },
  "signals": [
    {
      "code": "BUREAU_MISSINGNESS_INCREASE",
      "severity": "Amber",
      "evidence_id": "EV-2026-06-003"
    }
  ],
  "generated_at": "2026-07-01T10:00:00Z"
}
```
