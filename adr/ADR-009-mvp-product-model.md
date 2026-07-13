# ADR-009: Parent + Product-Extension Data Model (Reject Single Wide Table)

**Status:** Accepted
**Date:** 2026-07-13
**Deciders:** Principal Enterprise Data Architect, Model Risk Architect

## Context
The prior schema iteration used one wide account/snapshot table for all products, with product-specific fields (utilization, tenor, amortization) sitting as nullable columns regardless of product. The brief explicitly flags this as "too simplistic" and requires common parent structures with revolving and term-loan extensions.

## Decision
`dim_account` and `fact_account_snapshot_monthly` are common parent structures at a product-agnostic grain (one row per account / one row per account per month). Product-specific mechanics live in `dim_account_revolving_ext` / `dim_account_termloan_ext` and `fact_revolving_behaviour_monthly` / `fact_termloan_behaviour_monthly`, joined 1:1 to the parent primary key, populated only for the applicable product family.

## Consequences
- Positive: a query against `fact_account_snapshot_monthly` alone (balance, status, DPD) is product-agnostic and clean; a query needing utilization explicitly opts into the revolving extension and gets no row (not a null) for term loans — correctly modeling "not applicable" versus "missing."
- Positive: adding Speedy Loan-specific fields in the enterprise build (e.g., a new prepayment-penalty structure) never touches the Credit Card extension table or any common-grain query.
- Positive: this is the concrete proof point for Principle 1.11 (product-specific extensions under a common semantic model) and directly enables the MVP-to-enterprise extension claim in `MVP_VS_ENTERPRISE.md` without schema rewrite.
- Negative: every product-aware query requires an explicit join decision (which extension, if any) — more upfront query authoring discipline than a flat table, judged a worthwhile trade for correctness.

## Alternatives considered
1. **Single wide table, nullable product-specific columns** — rejected: this is the exact failure mode the brief calls out; cannot express "not applicable," invites incorrect joins, and does not scale cleanly to a fourth product with different mechanics.
2. **Fully generic EAV (entity-attribute-value) model** — rejected: defeats strong typing and query performance, and makes the schema unreadable to the Model Risk Committee, who need to reason about it directly during governance review.
3. **Separate account/snapshot tables per product (no shared parent)** — rejected: loses the common semantic model requirement; every cross-product monitoring query (e.g., "population count across all products") would require a UNION of three structurally different tables instead of one grain-consistent parent table.
