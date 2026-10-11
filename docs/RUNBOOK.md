# MilMed Connect PitchBook — Prompt Runbook — Order of Operations

Run these in order. Each prompt is a separate session with your Grok bot. Wait for the PR to merge before starting the next.

## Step 0 — API key discovery check (run on the iMac, not the bot)

**When:** Before anything else. Takes two minutes.

**Where:** Terminal on the MILMED Connect iMac, from the repo root.

**What:** Follow docs/API_KEY_CHECK.md — run the inline Python script (or `python -m api.config_check` once Phase 1 builds it). It reports which source supplies each key (Anthropic, OpenAI, Jev) without printing values, and tells you which Phase 2 layers can start immediately.

**Why first:** The result determines whether Phase 2 starts with the LLM layer, the Jev layer, both, or rule-based only. Knowing this before building saves rework.

## Step 1 — Phase 1 prompt

**When:** After Step 0.

**Where:** Paste into your Grok bot as the opening prompt of a fresh session.

**What:** The Phase 1 prompt from the conversation (MTEC crawler, SBIR.gov pull, schema, Splink dedup, rule-based scoring, FastAPI endpoints, API key discovery wired in api/config.py). No LLM, no Jev.

**Done when:** PR opened, all 13.5k MTEC profiles in Postgres, SBIR awards pulled, search API responding.

## Step 2 — Phase 2 prompt

**When:** After Phase 1's PR merges.

**What:** The Phase 2 prompt (Jev scoring layer, LLM scoring with evidence citations, USAspending + CDMRP ingestion, cross-source Splink, company profile pages). Jev degrades gracefully if no key found; LLM layer activates only if an Anthropic/OpenAI key was found in Step 0.

**Done when:** PR opened, both scoring layers calibrated against the 50 hand-labeled companies, cross-source matches visible.

## Step 3 — Phase 3 prompt

**When:** After Phase 2's PR merges.

**What:** The Phase 3 prompt from docs/PHASE3_PROMPT.md (MCP server with six tools, React dashboard, daily cron refresh on the iMac, email/Slack alerts, review queue).

**Done when:** PR opened, all three demo queries answer in under five seconds through both dashboard and MCP.

## Step 4 — Phase 4 prompt

**When:** After Phase 3's PR merges.

**What:** The Phase 4 prompt from docs/PHASE4_PROMPT.md (TechLink ingestion with company matching, relationship graph, proposal-language generator, performance pass, exports, backup cron).

**Done when:** PR opened, TechLink matches surfaced, graph view renders, proposal drafts reviewable.

## Step 5 — Demo

**When:** After Phase 4 merges.

**What:** Walk Eric and Kirtly through the three demo queries, the one-pager (docs/ONE_PAGER.md), and the relationship graph. The pitch: this replaces $20k+/year of PitchBook for military-medical coverage, runs locally on their iMac, and answers cross-source questions in seconds.

## Notes

- Each phase prompt is also saved in the repo: docs/PHASE3_PROMPT.md, docs/PHASE4_PROMPT.md, docs/API_KEY_CHECK.md.
- If a phase's PR has review comments, address them in the same bot session before moving on.
- The API key check (Step 0) is the only step that runs on the iMac directly rather than through the bot.