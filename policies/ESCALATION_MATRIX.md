# Escalation Matrix

**Audience:** all agents (structural triggers), Model Risk Committee.

| Trigger | Severity | Escalation path | SLA | Bypass possible? |
|---|---|---|---|---|
| PSI/CSI Amber, tier-1 model | Medium | Standard investigation → critic review → committee | Investigation opened within 5 business days (POL-MMP-3.2) | No |
| PSI/CSI Red, tier-1 model | High | Standard investigation + immediate Teams notification to MRC chair | Same week | No |
| Any B Score `mob_at_observation < 6` | Critical | Immediate investigation, `severity = critical`, bypasses normal monthly-cycle batching | Same day | No — structural, cannot be batched or deferred |
| Model version mismatch (`fact_model_score_event` references a non-active deployment) | High | Immediate investigation + notify Governance Agent's model owner contact | Same day | No |
| Data-quality `fail` on tier-1 model source table completeness | High | Investigation (`trigger_type = dq_failure`) + Teams notification | Same day | No |
| Two consecutive Assurance/Critic rejections of the same finding | Escalation event (not a metric severity) | Direct human escalation, bypasses further automated critic/investigation cycling | Immediate | N/A — this *is* the bypass mechanism, by design |
| Recommendation type = `recalibration` or `retirement` | Always requires committee | No analyst-only approval path exists for lifecycle-changing recommendations | Standard committee cycle | No |
| Agent override rate exceeds `AGENT_OVERRIDE_RATE_MAX` threshold | Governance review trigger | Triggers Assurance/Critic Agent quality review of the relevant agent's recent findings | Next scheduled batch eval | No |
| Remediation action past `target_completion_date` | Operational | Teams escalation to accountable owner + MRC chair cc | Within 1 business day of the missed date | No |

## Principle
No agent can independently decide to skip an escalation row above — escalation logic lives in `dim_governance_threshold` (`rule_type = escalation`) and in structural code paths (e.g., the two-rejection rule is enforced in the Assurance Agent's own contract, not left to its discretion per interaction).
