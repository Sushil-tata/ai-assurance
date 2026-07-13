# Policy Test Cases

**Audience:** whoever builds/validates the `dim_governance_threshold` and RAG logic.

| ID | Check | Expected |
|---|---|---|
| POL-01 | PSI = 0.09 for CC Behaviour Score as of 2026-06-30 | `rag_status = green` |
| POL-02 | PSI = 0.263 for CC Behaviour Score as of 2026-06-30 (post seasonal band widening) | `rag_status = amber` (not red — tests the effective-dated threshold row, not a hardcoded 0.25 cutoff) |
| POL-03 | PSI = 0.263 evaluated as of 2026-03-01 (before the seasonal widening took effect) | `rag_status = red` (same value, different date, different governed threshold in effect — proves policy-as-code effective-dating works) |
| POL-04 | Sample size = 400 for a metric requiring minimum 1,000 | `statistical_reliability_flag = false`, RAG still computed but flagged |
| POL-05 | B Score eligibility check for `mob=5` | `eligibility_result = false` |
| POL-06 | B Score eligibility check for `mob=6` | `eligibility_result = true` |
| POL-07 | Recommendation type = `recalibration` | `requires_committee_approval = true` (always, no analyst-only path) |
| POL-08 | Free-text policy search for "what happens if PSI is red" | Returns POL-MMP-3.3 with exact clause text |
| POL-09 | Free-text policy search for an unrelated topic (e.g., "vacation policy") | Returns "not found in indexed policy library," not a fabricated answer |

## Threshold effective-dating regression test (critical)
POL-02 and POL-03 together are the single most important pair in this file: they confirm the exact same metric value produces different RAG outcomes depending on which threshold version was in effect on the evaluation date — proving ADR-007 (policy-as-code with effective dating) actually works end-to-end, not just as a documented intention.
