# Products

## Metadata
- Document Owner: AI Assurance Product and Governance Team
- Version: 1.0
- Effective Date: July 2026
- Review Frequency: Quarterly
- Status: Draft for Training Demonstration

## Purpose
Define product context for monitoring, investigation, and governance interpretation.

## Scope
Covers Credit Card, Speedy Cash, and Speedy Loan in MVP and enterprise-reference operating modes.

## Product Types
### Credit Card
- Type: Revolving product.
- Balance behavior: Dynamic revolving balance and payment utilization patterns.
- Typical monitoring focus: utilization drift, payment hierarchy effects, line assignment changes, delinquency migration.

### Speedy Cash
- Type: Revolving product.
- Balance behavior: Short-cycle revolving exposures with frequent draw and repay behavior.
- Typical monitoring focus: draw frequency shift, cycle-level payment volatility, bureau refresh quality, delinquency transition speed.

### Speedy Loan
- Type: Term loan product.
- Balance behavior: Contracted amortization schedule with fixed or structured installment behavior.
- Typical monitoring focus: installment adherence, prepayment and restructuring effects, term-stage delinquency transitions.

## Revolving Versus Term Monitoring Implications
- Revolving products: utilization and payment-pattern variables are highly meaningful.
- Term loans: installment schedule adherence and term-stage features are more meaningful than utilization-ratio patterns.
- Cross-product comparisons should normalize for product mechanics before interpreting drift or performance changes.

## Product-Specific Field Relevance
| Field Group | Credit Card | Speedy Cash | Speedy Loan | Notes |
|---|---|---|---|---|
| Credit line and utilization | Meaningful | Meaningful | Not meaningful | Term loans do not revolve against a dynamic line.
| Installment schedule adherence | Limited | Limited | Meaningful | Core term-loan repayment behavior indicator.
| Revolving payment ratio | Meaningful | Meaningful | Not meaningful | Not applicable to fixed installment design.
| Remaining tenor | Limited | Limited | Meaningful | Strong term-loan context variable.
| Delinquency bucket transitions | Meaningful | Meaningful | Meaningful | Used for collections and C Score monitoring.
| Bureau feature completeness | Meaningful | Meaningful | Meaningful | Data quality issue affects all products.

## Related Documents
- [BUSINESS_GLOSSARY.md](BUSINESS_GLOSSARY.md)
- [../models/MODEL_INVENTORY.md](../models/MODEL_INVENTORY.md)
- [../models/C_SCORE.md](../models/C_SCORE.md)

## Copilot Usage
Use this file to resolve product mechanics before interpreting signals, building hypotheses, or drafting remediation options.