# MilMed Connect PitchBook — Step-by-Step Instructions

Everything you need, in order. Each step says **when** to do it, **where** to do it, and **exactly what to paste**.

---

## STEP 0 — API key check (on the iMac, before anything else)

**When:** Before starting any bot session. Takes two minutes.

**Where:** Terminal on the MILMED Connect iMac.

**What to run:**

```bash
cd ~
git clone https://github.com/seantei/milmed-connect.git
cd milmed-connect
```

Then run the inline script from `docs/API_KEY_CHECK.md` (the Python block under "How to run"). It prints which source supplies each key — `.env`, shell profile, Keychain, or CLI config — without ever showing values, and tells you which Phase 2 layers can start immediately.

**Result:** You know whether Phase 2 starts with the LLM layer, Jev, both, or rule-based only.

---

## STEP 1 — Phase 1 (fresh Grok bot session)

**When:** After Step 0.

**Where:** Open a new Grok bot session. Paste this as the opening prompt:

```
Build Phase 1 of MilMed Connect PitchBook from the repo at github.com/seantei/milmed-connect. This runs locally on the MILMED Connect iMac — no Docker, no containers, no compose files. Postgres 16 and Redis install directly via Homebrew, Python 3.12 in a venv, Node 20+ for the frontend. Follow the README architecture exactly: ingest, normalize, score, api, mcp, dashboard folders.

Build the MTEC crawler first — httpx and BeautifulSoup4, pulling all 13,506 profiles into Postgres. Rate-limit at one request per two seconds, identify the bot in the User-Agent, and checkpoint progress so a crash doesn't restart from page one. Then pull SBIR.gov DoD medical awards via the official API. Add the SQL schema from docs, Splink dedup within MTEC, and rule-based scoring from tags and keywords — no LLM, no Jev. Expose a basic FastAPI with /companies, /companies/{id}, and /search endpoints.

Wire up API key discovery in api/config.py: check the project .env first, then shell profile exports, then macOS Keychain via the security command. Log which source supplied each key — never the value itself. Fail loudly naming the missing key and where to put it. Use whatever keys already exist on the iMac; create none.

When Phase 1 runs end to end, open a pull request and stop.
```

**Done when:** PR opened, all 13.5k MTEC profiles in Postgres, SBIR awards pulled, search API responding.

**Then:** Review the PR, merge it, and only then start Step 2.

---

## STEP 2 — Phase 2 (new Grok bot session)

**When:** After Phase 1's PR merges.

**Where:** Open a new Grok bot session. Paste this as the opening prompt:

```
Build Phase 2 of MilMed Connect PitchBook from the repo at github.com/seantei/milmed-connect, continuing from the merged Phase 1 branch. No Docker — everything stays local on the MILMED iMac with the existing Postgres and Redis. API keys continue to be discovered from existing machine storage (.env → shell profile → macOS Keychain) via api/config.py.

First, verify the API key discovery from api/config.py works: run it and log which source supplies each key without printing values. If an Anthropic or OpenAI key is found, the LLM scoring layer activates; if not, rule-based scoring continues and the LLM layer waits.

Then add the Jev layer in score/jev_score.py using the official TypeSafe Python SDK. Jev handles the high-volume decisions: rubric score questions for each MTEC focus area batched 7–10 per call, noul questions for the entity-resolution borderline queue and contradiction flags, and choice questions for source routing. If no JEV_API_KEY is found, degrade gracefully to rule-based scoring — Jev is an accelerator, not a dependency.

Add the LLM scoring layer in score/llm_score.py using the prompt from docs/LLM_SCORING_PROMPT.md: evidence citations, plain-English summaries, and contradiction flags for what Jev can't reason about. Calibrate both layers against the 50 hand-labeled companies in docs, tracking precision and recall.

Then ingest USAspending DoD medical contracts via the bulk-download API filtered to CFDA 12.420, and scrape CDMRP funding-history pages. Run cross-source Splink matching across MTEC, SBIR, and USAspending, and build company profile pages with funding timelines.

When Phase 2 is complete, open a pull request and stop.
```

**Done when:** PR opened, both scoring layers calibrated, cross-source matches visible.

**Then:** Review the PR, merge it, and only then start Step 3.

---

## STEP 3 — Phase 3 (new Grok bot session)

**When:** After Phase 2's PR merges.

**Where:** Open a new Grok bot session. Paste this as the opening prompt:

```
Build Phase 3 of MilMed Connect PitchBook from the repo at github.com/seantei/milmed-connect, continuing from the merged Phase 2 branch. No Docker — everything stays local on the MILMED Connect iMac with the existing Postgres and Redis. API keys continue to be discovered from existing machine storage (.env → shell profile → macOS Keychain) via api/config.py.

Build the MCP server (mcp/, FastMCP) with six tools: search_companies, score_company, get_funding_history, find_dual_use_matches, get_new_matches, get_solicitation_alerts. Test each against the three demo queries in docs/BUILD_PLAN.md.

Build the React dashboard (dashboard/, React 18 + Vite + Tailwind + Recharts): search page with focus-area and capability-tag filters plus a dual-use score slider; company profile pages; funding map; alerts panel; entity-resolution review queue.

Daily refresh via cron on the iMac (not GitHub Actions): 0 2 * * * cd /path/to/milmed-connect && .venv/bin/python -m ingest.run_all >> logs/ingest.log 2>&1. ingest.run_all re-crawls MTEC (content-hash change detection), pulls new SBIR/USAspending records, re-scores only what changed, and logs a summary.

Email (SMTP) and optional Slack alerts for companies scoring above 80. Suppression list respected.

Acceptance test: all three demo queries answer in under five seconds through both the dashboard and the MCP server.

When Phase 3 is complete, open a pull request and stop.
```

**Done when:** PR opened, all three demo queries under five seconds through both dashboard and MCP.

**Then:** Review the PR, merge it, and only then start Step 4.

---

## STEP 4 — Phase 4 (new Grok bot session)

**When:** After Phase 3's PR merges.

**Where:** Open a new Grok bot session. Paste this as the opening prompt:

```
Build Phase 4 of MilMed Connect PitchBook from the repo at github.com/seantei/milmed-connect, continuing from the merged Phase 3 branch. No Docker — everything stays local on the MILMED Connect iMac with the existing Postgres and Redis. API keys continue to be discovered from existing machine storage (.env → shell profile → macOS Keychain) via api/config.py.

TechLink ingestion (ingest/techlink.py): keyword scrape of ~1,945 medical technologies across 100+ DoD/VA labs. No API — same rate-limiting and checkpointing as MTEC. Match technologies to companies (inventor-founded, licensing-contact matches) using Jev noul questions. Surface as a "Lab-origin technologies" section on profiles.

Relationship graph (graph/, NetworkX): node types are companies, DoD PIs, labs, programs, awards. Edges connect them. Dashboard view with react-force-graph or D3. Killer query: shortest path from a company to a combat casualty care PI at USAMRDC.

Proposal-language generator (proposals/): given a company profile and an MTEC solicitation, draft the technical approach, military relevance, dual-use, and team sections using the LLM layer. Include a human-review disclaimer and version history in the DB.

Performance pass: indexes on focus_area, dual_use_score, seeking_investment. CSV and PDF exports. pg_dump backup cron alongside the ingest cron.

When Phase 4 is complete, open a pull request and stop.
```

**Done when:** PR opened, TechLink matches surfaced, graph view renders, proposal drafts reviewable.

**Then:** Review the PR, merge it, and move to Step 5.

---

## STEP 5 — Demo (with Eric and Kirtly)

**When:** After Phase 4 merges.

**Where:** On the iMac, with Eric and Kirtly present.

**What to show:**

1. The three demo queries from docs/BUILD_PLAN.md — run each live through the dashboard and the MCP server.
2. The one-pager at docs/ONE_PAGER.md as the leave-behind.
3. The relationship graph — the shortest-path query from a company to a USAMRDC PI.

**The pitch:** This replaces $20k+/year of PitchBook for military-medical coverage, runs locally on their iMac, and answers cross-source questions in seconds that they currently research manually across five-plus sites.

---

## Alternative: one session, all phases

If you'd rather not open five sessions, paste the contents of `docs/MASTER_PROMPT.md` as the opening prompt of a single Grok bot session. It contains every phase and tells the bot to open a PR and stop after each one — same sequence, one conversation.

## Notes

- Every phase prompt is also saved as its own file in the repo: docs/PHASE3_PROMPT.md, docs/PHASE4_PROMPT.md, docs/API_KEY_CHECK.md.
- If a phase's PR has review comments, address them in the same bot session before moving on.
- The API key check (Step 0) is the only step that runs on the iMac directly rather than through the bot.
- The GitHub repo URL stays github.com/seantei/milmed-connect; the project name is MilMed Connect PitchBook.