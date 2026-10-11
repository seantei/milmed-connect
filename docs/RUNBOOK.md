# MilMed Connect PitchBook — Prompt Runbook

**Authoritative steps are in docs/INSTRUCTIONS.md.** This file is a short checklist.

Run these in order. Each bot session is fresh. Wait for the PR to merge and human review before starting the next phase.

## Step 0 — Foundations (on the iMac + with Eric and Kirtly)

**When:** Before any bot session.

**What:** Follow Step 0 in docs/INSTRUCTIONS.md. Verify source access, agree on scoring, approve three demo questions with expected answers, create labeled sets, document hardware. Optional: run the key check from docs/API_KEY_CHECK.md.

**Done when:** Approved Phase 0 artifacts exist in named files (schema, scoring contract, source findings, demo questions/answers, labeled datasets). Human approval recorded.

## Step 1 — Phase 1 (thin end-to-end)

**When:** After Step 0 artifacts are approved.

**What:** Paste the Phase 1 prompt from docs/INSTRUCTIONS.md. Bot must read Phase 0 artifacts first and stop if any are missing.

**Done when:** PR opened. Two sources ingested, search returns results with evidence links, conservative matching passes held-out set, backup restoration demonstrated. Eric and Kirtly see a useful result.

## Step 2 — Phase 2 (improve against feedback)

**When:** After Phase 1 merges and feedback is incorporated.

**What:** Paste the Phase 2 prompt from docs/INSTRUCTIONS.md.

**Done when:** Rules-only path passes all core tests. If an approved usable AI provider is enabled, its evaluation also meets the agreed criteria; otherwise it is explicitly marked disabled. Demo questions answer with cited evidence. No double-counting of awards.

## Step 3 — Phase 3 (platform and formal demo)

**When:** After Phase 2 merges.

**What:** Paste the Phase 3 prompt from docs/INSTRUCTIONS.md (or docs/PHASE3_PROMPT.md if updated).

**Done when:** Formal demo prepared. Bot opens PR. Eric and Kirtly approve usefulness — the bot does not declare the meeting delivered on their behalf.

## Step 4 — Extension (separately estimated)

Only after the core product proves useful. See docs/INSTRUCTIONS.md.

## Notes

- Every phase prompt must include: "Read the approved Phase 0 artifacts first. If required artifacts or human approvals are missing, report the missing items and stop; do not invent approvals or labels."
- AI scoring is optional. The rules-only path must always work.
- See docs/BUILD_PLAN.md for full acceptance criteria.
