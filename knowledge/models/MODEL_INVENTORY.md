# Model Inventory

## Metadata
- Document Owner: AI Assurance Product and Governance Team
- Version: 1.0
- Effective Date: July 2026
- Review Frequency: Quarterly
- Status: Draft for Training Demonstration

## Purpose
Provide a concise, retrieval-friendly inventory of in-scope and architecture-reference models.

## Scope
Includes Credit Card, Speedy Cash, and Speedy Loan models across A Score, B Score, and C Score families.

## Inventory
| Model ID | Model Name | Product | Family | Subtype | Observation Unit | Eligibility Rule | Performance Horizon | Monitoring Frequency | Status | Materiality |
|---|---|---|---|---|---|---|---|---|---|---|
| M-CC-A-01 | CC Application Score | Credit Card | A Score | Acquisition | Application | Approved applications | 3, 6, 12 months forward | Monthly | Architecture Reference | Medium |
| M-CC-B-01 | CC Behaviour Score | Credit Card | B Score | Post-booking | Account-Month | MOB >= 6 | 3, 6, 12 months forward | Monthly | MVP Active | High |
| M-CC-C0-01 | CC Bucket 0 C Score | Credit Card | C Score | Bucket 0 | Account-Month | Active in Bucket 0 | 1 month transition and 3-month trend | Monthly | Architecture Reference | Medium |
| M-CC-C1-01 | CC Bucket 1 C Score | Credit Card | C Score | Bucket 1 | Account-Month | Active in Bucket 1 | 1 month transition and 3-month trend | Monthly | Architecture Reference | Medium |
| M-SPC-A-01 | SPC Application Score | Speedy Cash | A Score | Acquisition | Application | Approved applications | 3, 6, 12 months forward | Monthly | Architecture Reference | Medium |
| M-SPC-B-01 | SPC Behaviour Score | Speedy Cash | B Score | Post-booking | Account-Month | MOB >= 6 | 3, 6, 12 months forward | Monthly | Architecture Reference | High |
| M-SPC-C0-01 | SPC Bucket 0 C Score | Speedy Cash | C Score | Bucket 0 | Account-Month | Active in Bucket 0 | 1 month transition and 3-month trend | Monthly | Architecture Reference | Medium |
| M-SPC-C1-01 | SPC Bucket 1 C Score | Speedy Cash | C Score | Bucket 1 | Account-Month | Active in Bucket 1 | 1 month transition and 3-month trend | Monthly | Architecture Reference | Medium |
| M-SPL-A-01 | SPL Application Score | Speedy Loan | A Score | Acquisition | Application | Approved applications | 3, 6, 12 months forward | Monthly | Architecture Reference | Medium |
| M-SPL-B-01 | SPL Behaviour Score | Speedy Loan | B Score | Post-booking | Account-Month | MOB >= 6 | 3, 6, 12 months forward | Monthly | Architecture Reference | High |
| M-SPL-C0-01 | SPL Bucket 0 C Score | Speedy Loan | C Score | Bucket 0 | Account-Month | Active in Bucket 0 | 1 month transition and 3-month trend | Monthly | Architecture Reference | Medium |
| M-SPL-C1-01 | SPL Bucket 1 C Score | Speedy Loan | C Score | Bucket 1 | Account-Month | Active in Bucket 1 | 1 month transition and 3-month trend | Monthly | Architecture Reference | Medium |

## Key Terms
- Family: A Score, B Score, or C Score model group.
- Subtype: Operational variant such as acquisition, post-booking, Bucket 0, or Bucket 1.
- Materiality: Relative governance criticality used for escalation and approval routing.

## Related Documents
- [A_SCORE.md](A_SCORE.md)
- [B_SCORE.md](B_SCORE.md)
- [C_SCORE.md](C_SCORE.md)
- [../business/PRODUCTS.md](../business/PRODUCTS.md)

## Copilot Usage
Use this inventory to resolve model identity, status, eligibility, and product context before policy checks or recommendation drafting.