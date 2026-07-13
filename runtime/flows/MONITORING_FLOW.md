# Monitoring Flow

## Input
- User request for model monitoring interpretation.
- Model and period context.

## Routing
1. AI Assurance Master receives request.
2. Route to Monitoring Specialist.
3. Monitoring Specialist calls GET_MODEL_MONITORING_SUMMARY.
4. If data quality signal is material, route to Data Quality Specialist.
5. If severity is Amber/Red, route to Investigation Specialist.

## Output
- Monitoring summary payload.
- Initial severity classification.
- Investigation trigger decision.

## Exceptions
- Tool timeout: retry once, then return blocked status.
- Missing period data: request alternate period or escalate.
- Model not found: return validation error and stop.
