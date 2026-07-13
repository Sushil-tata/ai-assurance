# Threshold Matrix

**Audience:** Governance Agent design owner, Model Risk Committee.
**Source of truth:** `dim_governance_threshold` (Domain 19). This file is the human-readable rendering of the currently effective rows relevant to the MVP model.

## CC Behaviour Score (`CC-BSCORE`) — effective 2026-04-01

| Metric | Green | Amber | Red | Min. sample | Policy clause |
|---|---|---|---|---|---|
| PSI | ≤ 0.10 | ≤ 0.28 | > 0.28 | 1,000 | POL-MMP-3.2 |
| CSI (per feature) | ≤ 0.10 | ≤ 0.25 | > 0.25 | 1,000 | POL-MMP-3.2 |
| Gini (relative to validation) | within 10% | 10–25% degradation | > 25% degradation | 1,000 mature, ≥30 bads | POL-MMP-4.2 |
| KS | within 10% | 10–25% degradation | > 25% degradation | 1,000 mature, ≥30 bads | POL-MMP-4.2 |
| Calibration O/E | [0.85, 1.15] | [0.7, 0.85) ∪ (1.15, 1.3] | outside | 100 per band | POL-MMP-4.3 |
| Bureau feature missingness | < 5% | 5–15% | > 15% | N/A | POL-MMP-5.1 |
| Sample-size sufficiency | N/A (flag, not RAG) | — | — | per-metric | POL-MMP-4.4 |

## Eligibility rules

| Rule | Definition | Policy clause |
|---|---|---|
| B Score MOB eligibility | `mob >= 6 AND account_status = 'active'` | POL-MMP-5.1 |
| A Score population | decisioned applications only (approved or declined; declined excluded from performance monitoring, included in approval-rate monitoring) | POL-MMP-2.1 |
| C Score bucket alignment | `delinquency_bucket_id` matches model subtype exactly | POL-MMP-5.2 |

## Note on the seasonal PSI band widening (referenced in Scenario 001)

The Amber upper bound for `CC-BSCORE` PSI was widened from 0.25 to 0.28 effective 2026-04-01, approved by the Model Risk Committee to account for known seasonal application-mix variation ahead of the year's second-quarter campaign cycle. This is itself a governed, versioned change (a new `dim_governance_threshold` row with its own `effective_start_date`, `approved_by`, and superseding the prior row's `effective_end_date`) — not a special case coded around, exactly as ADR-007 requires.

## Enterprise target
All twelve model inventory rows (`data/MODEL_INVENTORY_SCHEMA.md`) will carry their own threshold sets. Only `CC-BSCORE`'s is populated with live data in MVP.
