# AI Assurance Runtime Knowledge Base

## Metadata
- Document Owner: AI Assurance Product and Governance Team
- Version: 1.0
- Effective Date: July 2026
- Review Frequency: Quarterly
- Status: Draft for Training Demonstration

## Purpose
This knowledge base provides concise runtime grounding for Microsoft Copilot Studio capabilities used by AI Assurance.

## Scope
The files in this folder define product context, model-family definitions, monitoring policy, investigation standards, evidence handling, committee guidance, and practical external reference summaries.

## File Catalog
- [business/BUSINESS_GLOSSARY.md](business/BUSINESS_GLOSSARY.md): Canonical definitions used across all runtime documents.
- [business/PRODUCTS.md](business/PRODUCTS.md): Product types, revolving versus term distinctions, and monitoring implications.
- [models/MODEL_INVENTORY.md](models/MODEL_INVENTORY.md): Illustrative model inventory for MVP and enterprise-reference models.
- [models/A_SCORE.md](models/A_SCORE.md): Acquisition-only model family definition and monitoring rules.
- [models/B_SCORE.md](models/B_SCORE.md): Monthly post-booking model family with MOB >= 6 applicability.
- [models/C_SCORE.md](models/C_SCORE.md): Bucket 0 and Bucket 1 collections monitoring model family.
- [policies/MODEL_MONITORING_POLICY.md](policies/MODEL_MONITORING_POLICY.md): Required monitoring review areas and operating principles.
- [policies/THRESHOLD_MATRIX.md](policies/THRESHOLD_MATRIX.md): Synthetic Green/Amber/Red thresholds for demonstration.
- [policies/ESCALATION_MATRIX.md](policies/ESCALATION_MATRIX.md): Role-based escalation paths and response expectations.
- [policies/REMEDIATION_PLAYBOOK.md](policies/REMEDIATION_PLAYBOOK.md): Allowed remediation actions with triggers and closure criteria.
- [policies/HUMAN_APPROVAL_POLICY.md](policies/HUMAN_APPROVAL_POLICY.md): Human decision rights for material actions.
- [governance/INVESTIGATION_GUIDELINES.md](governance/INVESTIGATION_GUIDELINES.md): End-to-end investigation workflow with June 2026 scenario.
- [governance/EVIDENCE_GUIDELINES.md](governance/EVIDENCE_GUIDELINES.md): Evidence quality rules and evidence ledger requirements.
- [governance/COMMITTEE_GUIDELINES.md](governance/COMMITTEE_GUIDELINES.md): Committee pack format and reporting boundaries.
- [governance/MODEL_RISK_STANDARD.md](governance/MODEL_RISK_STANDARD.md): Practical internal model lifecycle and accountability standard.
- [references/RBI_MRM_SUMMARY.md](references/RBI_MRM_SUMMARY.md): Practical non-legal summary of model risk management themes.
- [references/MAS_FEAT_SUMMARY.md](references/MAS_FEAT_SUMMARY.md): Practical non-legal FEAT principles summary.
- [references/EBA_ML_IRB_SUMMARY.md](references/EBA_ML_IRB_SUMMARY.md): Practical non-legal ML governance themes for credit risk.
- [references/NIST_AI_RMF_SUMMARY.md](references/NIST_AI_RMF_SUMMARY.md): Govern, Map, Measure, Manage mapping for AI Assurance.

## What Belongs Here
- Retrieval-focused markdown grounding used at runtime by Copilot Studio.
- Stable definitions, operating rules, and policy summaries.
- Synthetic and illustrative content for demonstration.

## What Does Not Belong Here
- Architecture design documentation.
- Test plans and test execution artifacts.
- SQL scripts, source code, deployment scripts, or API contracts.
- Customer-level, confidential, or personally identifiable data.

## How Copilot Studio Should Use These Documents
- Use glossary and model files to normalize terminology.
- Use policy files to classify severity, escalation, remediation options, and approval needs.
- Use governance files to structure investigation outputs and evidence-backed findings.
- Use reference summaries only as practical context, not legal or regulatory advice.

## Copilot Usage
AI Assurance Orchestrator and specialist agents retrieve this folder as the primary runtime grounding source for monitoring interpretation, investigation drafting, governance checks, and committee-pack preparation.