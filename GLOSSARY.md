# Glossary

**Audience:** everyone. This file resolves ambiguity — every other document assumes these definitions.

## Products and accounts

| Term | Definition |
|---|---|
| Credit Card (CC) | Revolving unsecured product; balance fluctuates against a credit limit. |
| Speedy Cash (SPC) | Revolving unsecured cash-access product; same behavioural mechanics as CC (utilization, limit, drawdown) but distinct pricing/risk profile. |
| Speedy Loan (SPL) | Term loan; fixed original tenor, scheduled instalments, amortizing principal. |
| MOB | Months on book. Integer count of full months since `booking_date`. MOB 0 = booking month. |
| Booking date | Date the account was opened/disbursed. Distinct from `application_date`. |
| Vintage | The booking month/quarter cohort an account belongs to, used to align performance comparisons across accounts of different age. |

## Model families

| Term | Definition |
|---|---|
| A Score | Acquisition / application score. Observation point = application date. Predicts forward performance from booking. |
| B Score | Behaviour score. Observation point = monthly account snapshot. Requires MOB ≥ 6. Produced monthly per account. |
| C Score | Delinquency/collections score. Observation point depends on delinquency bucket (Bucket 0 = current/early, Bucket 1 = one cycle down, etc.). Outcomes: cure, remain, roll forward, NPL, charge-off, restructuring. |
| Bucket | Delinquency state bucket, e.g. Bucket 0 = 1–29 DPD, Bucket 1 = 30–59 DPD (bucket boundaries are configured per product in `dim_governance_threshold`, not hardcoded). |

## Temporal terms

| Term | Definition |
|---|---|
| Event time | The date/time the real-world thing happened (e.g., a payment was made). |
| Processing time | The date/time the record was loaded/computed by AI Assurance pipelines. |
| Effective time | The date range during which a dimension row is considered the "current truth" (SCD2). |
| System time | Database-level insert/update timestamp, used for pure technical audit, not for business logic. |
| Observation window | The point in time features are computed *as of* — must never include information from after that date (no leakage). |
| Performance window | The forward period (3/6/12 months) over which an outcome/label is measured. |
| Maturity date | `observation_date + performance_window`. Before this date, an outcome is **immature** and must not be used in any monitoring metric. |
| Censored observation | An account that closed, was paid off, or otherwise stopped being observable before its performance window matured — the label is incomplete, not "good." |
| Late-arriving data | A source record whose event time precedes its processing time by more than the expected pipeline latency (e.g., a bureau file reprocessed two weeks late). |
| Restatement | A correction to a previously published metric or label caused by late-arriving or corrected data, tracked with its own version, never overwriting history silently. |
| Back-scoring | Running a *current* model version against *historical* observation dates to build a comparison baseline — always logged as `run_type = 'backscore'`, never conflated with live scoring. |

## Monitoring terms

| Term | Definition |
|---|---|
| PSI | Population Stability Index — score/feature distribution shift between a reference and current population. |
| CSI | Characteristic Stability Index — PSI computed per input feature/characteristic rather than on the final score. |
| RAG status | Red/Amber/Green status assigned to a metric value by comparing it to threshold bands in `dim_governance_threshold`. |
| Statistical reliability flag | Marks a metric value as directional-only when the underlying sample size is below the minimum required for the stated confidence level. |
| Roll rate | Proportion of accounts in delinquency bucket *n* in month *t* that are in bucket *n+1* (or worse) in month *t+1*. |
| Cure rate | Proportion of delinquent accounts that return to current status within a defined window. |
| Redefault | An account that cured and subsequently re-entered delinquency within a defined observation window. |

## Governance and investigation terms

| Term | Definition |
|---|---|
| Trigger | The specific metric-breach or rule event that opens an investigation. |
| Hypothesis | A candidate explanation proposed by the Investigation Agent, tested against evidence, not assumed true. |
| Evidence item | A single, independently citable fact (metric value, row, tool output, policy clause) stored in the evidence ledger. |
| Finding | A conclusion the Investigation Agent believes is supported by evidence, subject to Assurance/Critic review before being shown to a human. |
| Root cause | The underlying driver of a finding, drawn from a controlled taxonomy (`dim_root_cause_taxonomy`), not free text. |
| Recommendation | A proposed remediation or monitoring action, always requiring human approval before execution. |
| Committee decision | The formal human/committee ruling that closes or escalates an investigation. |

## Agent terms

| Term | Definition |
|---|---|
| Orchestrator | The routing agent; decides which specialist agent handles a request, never performs domain reasoning itself. |
| Grounding | The requirement that every factual claim an agent makes trace to a tool output or evidence-ledger row. |
| Critic loop | The mandatory Assurance Agent review step between an Investigation Agent finding and human visibility. |
| Human-in-the-loop boundary | The explicit point past which no agent may proceed without a named human decision. |
