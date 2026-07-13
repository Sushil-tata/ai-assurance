# Investigation Summary Card Specification

## Title
AI Assurance Investigation Summary

## Fields
- Investigation ID
- Model ID
- Period
- Overall Severity
- Primary Cause
- Contributing Factors
- Unrelated Signals
- Confidence
- Recommended Next Action
- Evidence IDs

## Buttons
- View Evidence Package
- Route to Governance
- Request More Analysis

## Actions
- `View Evidence Package`: open evidence ledger detail.
- `Route to Governance`: create governance recommendation draft.
- `Request More Analysis`: generate follow-up investigation request.

## Validation Rules
- Card must include at least one evidence ID.
- Confidence must be High, Medium, or Low.
- Primary cause is mandatory before governance routing.
