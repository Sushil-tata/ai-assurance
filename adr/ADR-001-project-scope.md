# ADR-001: Project Scope Lock — AI Assurance, Credit Card Behaviour Score MVP

**Status:** Accepted
**Date:** 2026-07-13
**Deciders:** Principal Enterprise Data Architect, Model Risk Architect, Copilot Studio Solution Architect

## Context
AI Assurance could be scoped narrowly (one model, full depth) or broadly (all products/models, shallow depth). Prior attempts at similar internal tools failed by trying to demonstrate breadth before depth, producing a shallow schema that could not support a real investigation narrative.

## Decision
Lock MVP scope to **Credit Card Behaviour Score (B Score) only**, full depth: monthly scoring, MOB ≥ 6 eligibility, 3/6/12-month forward monitoring, one fully engineered investigation carried to committee decision. The underlying schema and agent design must generalize to all products and model families without redesign — see ADR-009.

## Consequences
- Positive: the MVP can be demonstrated end-to-end with no "and imagine the rest" gaps.
- Positive: the schema is validated against real complexity (temporal windows, product extensions) even though only one branch is populated with data.
- Negative: SPC and SPL, and A/C Score, are not demonstrated live — only structurally supported. Reviewers must be told this explicitly (see `SCOPE.md`).

## Alternatives considered
1. **Shallow-and-wide** (all products, minimal depth per product) — rejected: cannot demonstrate a real investigation, and risks recreating the "too simplistic" schema the brief explicitly rejects.
2. **Two products, one model family each** — rejected: doubles synthetic data and agent-testing effort for marginal narrative benefit; one deep example proves the pattern.
