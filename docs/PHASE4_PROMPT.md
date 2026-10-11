# Phase 4 Prompt — Differentiation Layer

Build Phase 4 of milmed-connect from the repo at github.com/seantei/milmed-connect, continuing from the merged Phase 3 branch. No Docker — everything stays local on the MILMED Connect iMac with the existing Postgres and Redis. API keys continue to be discovered from existing machine storage (.env → shell profile → macOS Keychain) via api/config.py.

## Goals

Phase 4 is what turns a good demo into a product Eric and Kirtly would actually pay to keep running. Three differentiators: TechLink technology-to-company matching, a relationship graph connecting companies to DoD PIs and labs, and a proposal-language generator for MTEC solicitations. This is the layer that makes the platform feel like it was built by someone who understands their business — not just scraped their data.

## 1. TechLink ingestion (ingest/techlink.py)

- Keyword scrape of the TechLink marketplace: ~1,945 medical technologies across 100+ DoD/VA labs. No API exists — use httpx + BeautifulSoup4 with the same rate-limiting and checkpointing as the MTEC crawler.
- Medical is TechLink's largest category, so prioritize it: filter by medical keywords (trauma, diagnostics, wearable, antimicrobial, etc.) before full crawl.
- Store each technology: title, description, lab, inventor(s), patent status, licensing contact, TRL, keywords.
- Match technologies to companies in the database: a TechLink technology whose inventor later founded a company, or whose licensing contact matches a company profile, is a high-value dual-use signal. Use Jev `noul` questions for borderline matches — "is this technology related to this company?" — with probability thresholds.
- Surface matched technologies on company profile pages as a "Lab-origin technologies" section.

## 2. Relationship graph (graph/)

Build a graph database view (NetworkX is fine for local scale; Neo4j optional if Eric wants it) with these node types:

- **Companies** — from the unified database
- **DoD PIs / investigators** — extracted from SBIR/CDMRP award records (PI names, institutions)
- **Labs** — from TechLink records (AFRL, ARL, NAMRU, USAMRDC, etc.)
- **Programs** — MTEC focus areas, CDMRP program areas, SBIR topics
- **Awards** — edges connecting companies to programs via award records

Edges: company↔award↔program, company↔PI, technology↔lab, PI↔lab (from award institution data).

Dashboard view: interactive graph (React + a library like react-force-graph or D3) where clicking any node shows its connections. The killer query: "show me the shortest path from this company to a combat casualty care PI at USAMRDC" — that's the introduction Eric would make manually.

## 3. Proposal-language generator (proposals/)

Given a company profile and an MTEC solicitation (scraped from mtec-sc.org solicitations page), generate draft proposal language:

- **Technical approach section**: map the company's products/expertise to the solicitation's requirements, citing specific evidence from the profile
- **Military relevance section**: tie to the focus area and any prior DoD funding
- **Dual-use section**: civilian traction as evidence the technology works outside the lab
- **Team section**: highlight military/clinical/regulatory background

Use the LLM layer for generation (this is reasoning work — Jev doesn't write). Include a disclaimer: AI-drafted, requires human review before submission. Store drafts in the DB with version history so Eric can iterate.

This is the feature that turns the platform from "nice database" into "this saves us a week per proposal."

## 4. Polish

- Performance pass: dashboard load times, MCP tool response times, database indexes on frequently filtered columns (focus_area, dual_use_score, seeking_investment)
- Export: CSV and PDF company profiles for offline sharing with DoD contacts
- Backup: simple pg_dump cron alongside the ingest cron
- README update: final architecture diagram, demo video script, and the one-pager from docs/ONE_PAGER.md as the leave-behind

## 5. Testing before PR

- TechLink: at least 500 medical technologies ingested with company matches surfaced
- Graph: all four node types populated, interactive view renders, path query returns results
- Proposals: generate drafts for two real MTEC solicitations against two real company profiles; Eric reviews for quality
- Run the three demo queries again — they should now include TechLink matches and graph context

## Done when

- [ ] TechLink ingestion running with company matching
- [ ] Relationship graph view live in dashboard
- [ ] Proposal-language generator producing reviewable drafts
- [ ] Performance pass complete, exports working, backup cron in place
- [ ] Open a pull request and stop

After Phase 4 merges, the platform is demo-ready for MILMED Connect. The conversation with Eric and Kirtly shifts from "can you build this?" to "when do we start using it?"