# SQL Actions (Fabric Warehouse Read Tools)

**Audience:** Fabric Warehouse / Copilot Studio SQL action builders.

## `get_monitoring_metric`
```sql
SELECT metric_code, calculation_run_id, run_date, reference_population_def,
       current_population_def, metric_value, sample_size_current,
       statistical_reliability_flag, rag_status, calculation_version
FROM fact_monitoring_metric
WHERE model_version_id = @model_version_id
  AND (@metric_code IS NULL OR metric_code = @metric_code)
  AND calculation_run_id = COALESCE(@calculation_run_id,
        (SELECT MAX(calculation_run_id) FROM fact_monitoring_metric WHERE model_version_id = @model_version_id));
```

## `get_segment_metric`
```sql
SELECT s.segment_dimension, s.segment_value, m.metric_code, m.metric_value,
       m.population_share_pct, m.sample_size_current, m.rag_status
FROM fact_segment_monitoring_metric m
JOIN dim_monitoring_segment s ON s.segment_id = m.segment_id
WHERE m.model_version_id = @model_version_id
  AND m.calculation_run_id = @calculation_run_id
  AND (@segment_dimension IS NULL OR s.segment_dimension = @segment_dimension);
```

## `get_dq_check_result`
```sql
SELECT rule_code, target_table, run_date, check_type, result_value,
       result_status, affected_row_count
FROM fact_data_quality_check
WHERE (@target_table IS NULL OR target_table = @target_table)
  AND (@rule_code IS NULL OR rule_code = @rule_code)
  AND run_id = COALESCE(@run_id, (SELECT MAX(run_id) FROM fact_data_quality_check));
```

## `get_feature_lineage` (enterprise; pre-computed for MVP demo)
```sql
SELECT fd.feature_name, fl.feature_value, fl.is_missing, fl.source_row_reference
FROM fact_feature_lineage fl
JOIN dim_feature_definition fd ON fd.feature_definition_sk = fl.feature_definition_sk
WHERE fl.score_event_id = @score_event_id;
```

## Guardrails applied to every SQL action
- All queries are parameterized (no string concatenation) — prevents injection regardless of what an LLM passes in.
- All queries have an implicit `TOP 1000` row cap unless the caller is an aggregation query (which returns O(1) rows by construction) — prevents an agent from accidentally pulling a bulk row export through a metric-lookup tool.
- No SQL action ever selects from `dim_customer` or `fact_bureau_tradeline` directly — those are excluded from the SQL action allow-list entirely at the connector configuration level, not just by convention.
