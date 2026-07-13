# Business Glossary

## Metadata
- Document Owner: AI Assurance Product and Governance Team
- Version: 1.0
- Effective Date: July 2026
- Review Frequency: Quarterly
- Status: Draft for Training Demonstration

## Purpose
Provide one unambiguous definition for key terms used by AI Assurance runtime workflows.

## Scope
Applies to all files under this knowledge base and all Copilot-generated monitoring, investigation, governance, and approval outputs.

## Terms
### A Score
Acquisition-only score generated at application date to estimate early future risk after booking.

### B Score
Monthly post-booking behavior score applied only when MOB >= 6.

### C Score
Collections score used for delinquency bucket transition monitoring, limited here to Bucket 0 and Bucket 1.

### MOB
Months on books, measured as complete months since booking date.

### DPD
Days past due for contractual payment obligations at observation date.

### PSI
Population Stability Index measuring distribution shift between current and reference populations.

### CSI
Characteristic Stability Index measuring shift in feature distributions between current and reference data.

### Gini
Discrimination metric derived from rank-ordering quality, often computed from AUC as 2 x AUC - 1.

### KS
Kolmogorov-Smirnov statistic measuring maximum separation between event and non-event cumulative distributions.

### AUC
Area under the ROC curve, representing discrimination ability across thresholds.

### Calibration
Alignment between predicted risk and observed outcome rate for a defined population and time window.

### O/E Ratio
Observed-to-Expected ratio comparing realized event rate to model-expected event rate.

### Maturity
Condition where a performance window is fully elapsed and outcomes are final enough for outcome monitoring.

### Censoring
Incompleteness of observed outcomes because full performance window has not elapsed or outcomes are interrupted.

### Vintage
Cohort defined by common origination or booking period for trend comparison over time.

### Roll Rate
Proportion of accounts transitioning from one delinquency bucket to a worse bucket over a defined interval.

### Cure
Transition from delinquent status to current or materially improved repayment status.

### Redefault
Return to delinquency after a prior cure within a defined follow-up period.

### Thin-file
Applicant or account profile with limited bureau or historical credit information.

### Policy-as-code
Executable representation of policy rules used for deterministic checks, traceability, and reproducibility.

### Evidence Ledger
Structured log of evidence items, sources, timestamps, tool calls, and supported conclusions.

### Investigation
Structured process to test hypotheses and establish evidence-backed findings for observed monitoring signals.

### Finding
Evidence-backed statement about what was observed and validated during investigation.

### Root Cause
Primary causal factor that best explains a material finding after testing counter-evidence.

### Recommendation
Proposed action based on findings, policy rules, and governance constraints.

### Human Approval
Recorded human decision required before material governance actions are executed.

### Remediation
Controlled action plan to address confirmed issues and close monitoring findings.

## Related Documents
- [PRODUCTS.md](PRODUCTS.md)
- [../models/B_SCORE.md](../models/B_SCORE.md)
- [../policies/MODEL_MONITORING_POLICY.md](../policies/MODEL_MONITORING_POLICY.md)

## Copilot Usage
All AI Assurance agents should treat this file as the canonical term source and resolve terminology conflicts in favor of these definitions.