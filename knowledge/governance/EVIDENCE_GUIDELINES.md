# Evidence Guidelines

## Metadata
- Document Owner: AI Assurance Product and Governance Team
- Version: 1.0
- Effective Date: July 2026
- Review Frequency: Quarterly
- Status: Draft for Training Demonstration

## Purpose
Define acceptable evidence standards for monitoring, investigation, and governance outputs.

## Scope
Applies to all evidence cited in AI Assurance findings, recommendations, and approval packs.

## Acceptable Evidence
- Deterministic metric extracts from approved computation workflows.
- Versioned policy outputs from policy-as-code checks.
- Data quality run logs and schema validation results.
- Controlled documents with traceable version and timestamp.

## Required Evidence Fields
- Evidence ID.
- Source.
- Table or document reference.
- Timestamp.
- Tool call.
- Metric.
- Policy version.
- Supported conclusion.

## Contradictory Evidence Rules
- Contradictory evidence must be logged, not omitted.
- Findings must state whether contradiction is resolved, unresolved, or partially resolved.
- Confidence level should be declared as High, Medium, or Low based on evidence completeness and consistency.

## Prohibited Claims
- Unsupported numeric statements.
- Unsupported policy interpretations.
- Conclusions without traceable evidence ID linkage.

## Evidence Ledger
The evidence ledger is the structured record joining evidence IDs, source lineage, timestamps, tool calls, metric outputs, and conclusion mapping used in assurance review.

## Related Documents
- [INVESTIGATION_GUIDELINES.md](INVESTIGATION_GUIDELINES.md)
- [COMMITTEE_GUIDELINES.md](COMMITTEE_GUIDELINES.md)
- [../policies/MODEL_MONITORING_POLICY.md](../policies/MODEL_MONITORING_POLICY.md)

## Copilot Usage
Use this file to validate whether generated findings are evidence-backed and to enforce complete evidence ledger fields in outputs.