# MilMed Connect PitchBook — Master Prompt — All Phases, One Session

Paste this as the opening prompt of a single Grok bot session. It contains everything the bot needs: the repo, the architecture, the no-Docker constraint, the iMac deployment target, the API key discovery rules, and all four phases with their acceptance criteria. The bot should execute phases sequentially, opening a PR after each phase and stopping for review before starting the next.

---

Build MilMed Connect PitchBook end to end from the repo at github.com/seantei/milmed-connect. This is a PitchBook-style dual-use medtech intelligence platform for MILMED Connect (Beaverton, OR) — a two-person firm that helps medtech and biotech companies win non-dilutive DoD funding through MTEC.

## Hard constraints (apply to every phase)

- **No Docker.** No containers, no compose files, nothing containerized. Postgres 16 and Redis install directly via Homebrew on the target machine.
- **Target machine: the MILMED Connect iMac** (macOS). Local-first. The data, database, API, dashboard, and MCP server all live on that one machine. Eric and Kirtly own everything outright.
- **API keys:** use whatever keys are already stored on the iMac. Discovery order: project `.env` (gitignored) → shell profile exports (`~/.zshrc` / `~/.zprofile`) → macOS Keychain via `security find-generic-password` → standard CLI config dirs (`~/.config/<tool>/`). Log which *source* supplied each key — never the value. Fail loudly naming the missing key and where to put it. Create no new keys.
- **Rate-limit crawlers** at one request per two seconds. Identify the bot in the User-Agent. Checkpoint all crawler progress so a crash doesn't restart from page one.
- **All source data is public and unclassified.** Respect the MTEC contact-email suppression list from day one.
- Follow the README architecture exactly: `ingest/`, `normalize/`, `score/`, `api/`, `mcp/`, `dashboard/` folders. Follow `docs/BUILD_PLAN.md`, `docs/DEPLOYMENT.md`, `docs/PHASE3_PROMPT.md`, and `docs/PHASE4_PROMPT.md` for phase-specific detail.

## Phase 1 — MVP (weeks 1–4)

1. Install Postgres 16 and Redis directly via Homebrew (`brew install` + `brew services start`). Create the `milmed` database and apply `normalize/schema.sql`.
2. Build the MTEC crawler (`ingest/mtec.py`, httpx + BeautifulSoup4): all 13,506 profiles into Postgres. Profiles live at `mtec-sc.org/life-sciences/{slug}` with description, products, expertise areas, key technologies, email, address, website, news. Directory paginates with `?page=N`; filters work (e.g., point-of-care diagnostics → 799 members).
3. Pull SBIR.gov DoD medical awards via the official API.
4. Build `api/config.py`: the key-discovery chain above, with source logging.
5. Splink dedup within MTEC. Rule-based scoring from tags and keywords only — no LLM, no Jev.
6. FastAPI with `/companies`, `/companies/{id}`, `/search` endpoints.
7. Open a PR and **stop**. Do not start Phase 2 until this merges.

## Phase 2 — Intelligence (weeks 5–8)

1. Verify key discovery from Phase 1: run the check from `docs/API_KEY_CHECK.md`. If an Anthropic or OpenAI key is found, the LLM layer activates; if not, it waits.
2. Build `score/jev_score.py` using the official TypeSafe Python SDK (github.com/typesafe-ai, docs.typesafe.ai). Jev handles: rubric `score` questions per MTEC focus area batched 7–10 per call; `noul` questions for the entity-resolution borderline queue (0.7–0.9 matches → human review) and contradiction flags; `choice` questions for source/tier routing. If no `JEV_API_KEY` is found, degrade gracefully to rule-based scoring — Jev is an accelerator, not a dependency.
3. Build `score/llm_score.py` using the LLM scoring prompt from the README: evidence citations, plain-English summaries, contradiction flags. This is the layer for what Jev can't reason about.
4. Ingest USAspending DoD medical contracts via the bulk-download API filtered to CFDA 12.420. Scrape CDMRP funding-history pages (no public API; Dimensions as fallback).
5. Cross-source Splink matching across MTEC, SBIR, and USAspending. Build company profile pages with funding timelines.
6. Calibrate both scoring layers against the 50 hand-labeled companies in docs. Track precision and recall.
7. Open a PR and **stop**.

## Phase 3 — Platform (weeks 9–12)

1. Build the MCP server (`mcp/`, FastMCP) with six tools: `search_companies`, `score_company`, `get_funding_history`, `find_dual_use_matches`, `get_new_matches`, `get_solicitation_alerts`. Test each against the three demo queries in `docs/BUILD_PLAN.md`.
2. Build the React dashboard (`dashboard/`, React 18 + Vite + Tailwind + Recharts): search page with focus-area and capability-tag filters plus a dual-use score slider; company profile pages; funding map; alerts panel; entity-resolution review queue.
3. Daily refresh via **cron on the iMac** (not GitHub Actions): `0 2 * * * cd /path/to/milmed-connect && .venv/bin/python -m ingest.run_all >> logs/ingest.log 2>&1`. `ingest.run_all` re-crawls MTEC (content-hash change detection), pulls new SBIR/USAspending records, re-scores only what changed, and logs a summary.
4. Email (SMTP) and optional Slack alerts for companies scoring above 80. Suppression list respected.
5. Acceptance test: all three demo queries answer in under five seconds through both the dashboard and the MCP server.
6. Open a PR and **stop**.

## Phase 4 — Differentiation

1. TechLink ingestion (`ingest/techlink.py`): keyword scrape of ~1,945 medical technologies across 100+ DoD/VA labs. No API — same rate-limiting and checkpointing as MTEC. Match technologies to companies (inventor-founded, licensing-contact matches) using Jev `noul` questions. Surface as a "Lab-origin technologies" section on profiles.
2. Relationship graph (`graph/`, NetworkX): node types are companies, DoD PIs, labs, programs, awards. Edges connect them. Dashboard view with react-force-graph or D3. Killer query: shortest path from a company to a combat casualty care PI at USAMRDC.
3. Proposal-language generator (`proposals/`): given a company profile and an MTEC solicitation, draft the technical approach, military relevance, dual-use, and team sections using the LLM layer. Include a human-review disclaimer and version history in the DB.
4. Performance pass: indexes on `focus_area`, `dual_use_score`, `seeking_investment`. CSV and PDF exports. `pg_dump` backup cron alongside the ingest cron.
5. Open a PR and **stop**.

## Demo (after Phase 4 merges)

Walk Eric and Kirtly through the three demo queries, the one-pager at `docs/ONE_PAGER.md`, and the relationship graph. The pitch: this replaces $20k+/year of PitchBook for military-medical coverage, runs locally on their iMac, and answers cross-source questions in seconds that they currently research manually across five-plus sites.

## Reference material in the repo

- `README.md` — architecture, scoring rubric, LLM prompt, tech stack
- `docs/BUILD_PLAN.md` — 12-week plan and demo queries
- `docs/DEPLOYMENT.md` — iMac install steps and key handling
- `docs/API_KEY_CHECK.md` — the key-discovery script
- `docs/PHASE3_PROMPT.md` / `docs/PHASE4_PROMPT.md` — phase-specific detail
- `docs/EXISTING_CLONES.md` — assessment of OpenPitch and OpenBook (borrow patterns, don't fork)
- `docs/ONE_PAGER.md` — the leave-behind for Eric and Kirtly
- `docs/RUNBOOK.md` — this sequence as separate prompts, if preferred

Execute Phase 1 now. Open the PR. Stop. Wait for review before Phase 2.