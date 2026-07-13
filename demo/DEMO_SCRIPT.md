# Demo Script

**Audience:** presenter (SAS Innovate or internal committee demo). Target runtime: 12–15 minutes plus Q&A.

## 0. Setup (before the room)
- Demo environment loaded with the full synthetic dataset (`data/SYNTHETIC_DATA_SPEC.md`), Scenario 001 fixtures included, monitoring run already showing `INV-2026-0031` as `open` (not yet investigated) so the audience sees the Investigation Agent work live.
- Teams window open to the AI Assurance bot; Power BI monitoring dashboard open in a second tab.

## 1. Open on the problem (90 seconds)
Show the current-state monitoring pack (a static Excel-style export) — one page, aggregate numbers, no explanation of *why* PSI moved. Say: "This is what a monitoring breach looks like today. Someone has to manually figure out why, by hand, from scratch, every time."

## 2. Show the trigger (60 seconds)
Switch to Power BI: `CC-BSCORE-v2.1` PSI Amber for June, `INV-2026-0031` already open. "This didn't wait for a person to notice — the Monitoring Agent opened this automatically the moment the metric breached threshold."

## 3. Ask the Orchestrator (live, in Teams) (3 minutes)
Type: *"Why is the CC Behaviour Score PSI amber this month?"*
Let the Orchestrator route to the Investigation Agent live. Narrate what's happening: hypothesis generation, tool calls to Monitoring/Data Quality/Governance agents, counter-evidence search. Show the finding as it's produced: three contributing factors, nine evidence items, one rejected hypothesis (stable bad rate).

## 4. Show the evidence ledger (2 minutes)
Open `demo/SAMPLE_EVIDENCE_LEDGER.md` (or its Power BI-rendered equivalent) — click through 2–3 evidence items to their source rows. "Every one of these traces to an actual table and row — nothing here is the model 'remembering' something plausible."

## 5. Show the critic rejection-and-fix, live if possible (2 minutes)
Either show a pre-recorded example or, ideally, live-inject a finding with a missing citation and show the Assurance Agent reject it by name ("sentence 3 has no linked evidence item") — this is the moment that differentiates AI Assurance from "just an LLM summarizing data."

## 6. Show the committee approval card in Teams (2 minutes)
Show the Reporting Agent's assembled report and the adaptive card; click "Approve" as the demo committee member; show the investigation close with `DEC-2026-0014` recorded and a named approver.

## 7. Close on governance, not just automation (90 seconds)
"Nothing closed itself. A human approved it, with full evidence in front of them, in minutes instead of days — and every step of how we got there is queryable, forever." Point to `architecture/HUMAN_OVERSIGHT.md` and the evidence ledger as the artifacts that make this defensible to a regulator, not just impressive to an audience.

## 8. Roadmap teaser (30 seconds)
Show `MVP_VS_ENTERPRISE.md`'s scope table briefly — "this is one model, one product, proven deep. The schema underneath already supports twelve models across three products without a redesign."

## Fallback plan
If live agent calls are slow/unreliable in the room, pre-record steps 3–6 and narrate over the recording, but keep step 5 (critic rejection) as close to live as possible — it's the single most convincing moment and worth the risk.
