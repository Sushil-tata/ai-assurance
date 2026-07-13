# AI Assurance Runtime Assets

## Purpose
This folder contains source-controlled runtime implementation assets for Microsoft Copilot Studio.

## Scope
These files define executable Copilot behavior: instructions, tool contracts, schemas, routing flows, card specifications, and runtime samples.

## Structure
- [instructions/AI_ASSURANCE_MASTER.md](instructions/AI_ASSURANCE_MASTER.md): Master runtime behavior contract.
- [instructions/MONITORING_SPECIALIST.md](instructions/MONITORING_SPECIALIST.md): Monitoring specialist module.
- [instructions/DATA_QUALITY_SPECIALIST.md](instructions/DATA_QUALITY_SPECIALIST.md): Data quality specialist module.
- [instructions/GOVERNANCE_SPECIALIST.md](instructions/GOVERNANCE_SPECIALIST.md): Governance specialist module.
- [instructions/INVESTIGATION_SPECIALIST.md](instructions/INVESTIGATION_SPECIALIST.md): Investigation specialist module.
- [instructions/ASSURANCE_SPECIALIST.md](instructions/ASSURANCE_SPECIALIST.md): Assurance specialist module.
- [instructions/REPORTING_SPECIALIST.md](instructions/REPORTING_SPECIALIST.md): Reporting specialist module.
- [tools/GET_MODEL_MONITORING_SUMMARY.md](tools/GET_MODEL_MONITORING_SUMMARY.md): Tool contract.
- [tools/GET_INVESTIGATION_EVIDENCE.md](tools/GET_INVESTIGATION_EVIDENCE.md): Tool contract.
- [tools/GET_GOVERNANCE_CONTEXT.md](tools/GET_GOVERNANCE_CONTEXT.md): Tool contract.
- [tools/WRITE_GOVERNANCE_DECISION.md](tools/WRITE_GOVERNANCE_DECISION.md): Tool contract.
- [schemas/monitoring_summary.json](schemas/monitoring_summary.json): Runtime output schema.
- [schemas/investigation_request.json](schemas/investigation_request.json): Runtime request schema.
- [schemas/investigation_result.json](schemas/investigation_result.json): Runtime output schema.
- [schemas/evidence_package.json](schemas/evidence_package.json): Runtime evidence schema.
- [schemas/governance_recommendation.json](schemas/governance_recommendation.json): Runtime output schema.
- [schemas/committee_report.json](schemas/committee_report.json): Runtime output schema.
- [flows/MONITORING_FLOW.md](flows/MONITORING_FLOW.md): Monitoring routing flow.
- [flows/INVESTIGATION_FLOW.md](flows/INVESTIGATION_FLOW.md): Investigation routing flow.
- [flows/ASSURANCE_FLOW.md](flows/ASSURANCE_FLOW.md): Assurance routing flow.
- [flows/REPORTING_FLOW.md](flows/REPORTING_FLOW.md): Reporting routing flow.
- [adaptive_cards/APPROVAL_CARD.md](adaptive_cards/APPROVAL_CARD.md): Human approval card spec.
- [adaptive_cards/INVESTIGATION_SUMMARY_CARD.md](adaptive_cards/INVESTIGATION_SUMMARY_CARD.md): Investigation summary card spec.
- [samples/SAMPLE_MONITORING_RESPONSE.json](samples/SAMPLE_MONITORING_RESPONSE.json): Sample runtime monitoring payload.
- [samples/SAMPLE_INVESTIGATION.json](samples/SAMPLE_INVESTIGATION.json): Sample investigation payload.
- [samples/SAMPLE_COMMITTEE_REPORT.md](samples/SAMPLE_COMMITTEE_REPORT.md): Sample committee report output.

## Runtime Rules
- Keep content implementation-focused and retrieval-friendly.
- Keep instructions and schemas consistent.
- Validate JSON and links before commit.
- Do not store confidential or customer-level data in samples.