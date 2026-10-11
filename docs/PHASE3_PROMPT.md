# Phase 3 Prompt — Platform Layer

Build Phase 3 of milmed-connect from the repo at github.com/seantei/milmed-connect, continuing from the merged Phase 2 branch. No Docker — everything stays local on the MILMED Connect iMac with the existing Postgres and Redis. API keys continue to be discovered from existing machine storage (.env → shell profile → macOS Keychain) via api/config.py.

## Goals

Turn the scored, cross-source database into a platform Eric and Kirtly can actually use every day: a React dashboard, an MCP server for AI-agent queries, and a daily cron refresh that keeps data fresh without manual intervention.

## 1. MCP server (mcp/)

Build with FastMCP. Expose these tools:

- `search_companies(query, filters)` — full-text + tag/focus-area search across the unified database
- `score_company(company_id)` — return the full scored profile: focus-area scores, dual-use, regulatory, readiness, evidence, flags, confidence
- `get_funding_history(company_id)` — all awards, contracts, and technologies across MTEC, SBIR, CDMRP, USAspending
- `find_dual_use_matches(criteria)` — e.g., "point-of-care diagnostics + hemorrhage control + SBIR award + no MTEC membership"
- `get_new_matches(since_date)` — companies newly scored above threshold since a date
- `get_solicitation_alerts()` — new MTEC solicitations matching focus areas

The MCP server should run alongside the FastAPI service. Test each tool against the demo queries in docs/BUILD_PLAN.md before opening the PR.

## 2. React dashboard (dashboard/)

- React 18 + Vite + Tailwind + Recharts
- Search page: query box, focus-area filters, capability-tag filters, dual-use score slider, "seeking investment" toggle
- Company profile pages: description, products, expertise, key technologies, funding timeline (all sources merged), score breakdown with evidence citations, contradiction flags
- Funding map: companies plotted by location, color-coded by top focus area
- Alerts panel: new high-score companies, new solicitations
- Review queue: borderline entity-resolution matches (0.7–0.9) for human adjudication

## 3. Daily refresh via cron (on the iMac)

Not GitHub Actions — the data lives locally. Add a cron entry:

`0 2 * * * cd /path/to/milmed-connect && .venv/bin/python -m ingest.run_all >> logs/ingest.log 2>&1`

`ingest.run_all` should:

- Re-crawl MTEC (detect new/changed profiles via content hash)
- Pull new SBIR awards since last run
- Pull new USAspending contracts since last run
- Re-score only new or changed records (Jev + LLM layers)
- Write a summary to logs/ingest.log: records added, changed, rescored, errors

## 4. Alerts

- Email alerts (SMTP) for new companies scoring above 80 on any focus area
- Optional Slack webhook for the same
- Suppression list respected: never alert on opted-out MTEC contacts

## 5. Testing before PR

Run the three demo queries from docs/BUILD_PLAN.md through both the dashboard and the MCP server:

1. Companies with point-of-care diagnostics + hemorrhage control + SBIR awards but no MTEC membership
2. CDMRP-funded companies currently seeking investment
3. Dual-use wearables: operational medicine tags + civilian FDA clearance + active clinical trials

Record response times. Target: under five seconds for each.

## Done when

- [ ] MCP server exposes all six tools and passes the demo queries
- [ ] Dashboard renders search, profiles, funding map, alerts, review queue
- [ ] Cron job runs nightly and logs correctly
- [ ] Alerts fire for test records
- [ ] Open a pull request and stop

Phase 4 (TechLink ingestion, relationship graph, proposal-language generator) comes after this merges.