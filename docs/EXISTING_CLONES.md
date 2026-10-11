# Existing Open-Source PitchBook Clones — Integration Assessment

Two real open-source PitchBook-style projects exist on GitHub. Here's what each does, what MilMed Connect PitchBook can borrow from them, and the honest verdict on integration.

## OpenPitch (github.com/Avierovich/openpitch) — MIT, ~10 stars, v0.1.3

**What it is:** An open, real-time intelligence layer for AI startups. Mines podcasts, news, and SEC EDGAR filings; extracts metrics with LLMs; reconciles conflicting sources into consensus ranges with confidence scores and contradiction flags; publishes daily via GitHub Actions. Ships an MCP server, a static dashboard, and an event feed. No API key needed — the data is committed to the repo as plain JSON.

**What MilMed Connect PitchBook can borrow:**

- **Confidence-scored, sourced figures** — every number carries its source, a confidence score, and a tracked history. This is the right trust model for dual-use discovery: Eric and Kirtly should see *why* a company scored 84, not just the number.
- **Contradiction detection** — when sources disagree, surface a consensus range + flag, don't silently pick one. Already planned in our LLM scoring layer; OpenPitch proves the pattern works.
- **MCP-native, zero-install design** — `uvx openpitch-mcp` works in any agent in under a minute. Our MCP server should follow the same ergonomics.
- **Daily pipeline as git commits** — the repo *is* the database; freshness is visible in git history. We use cron on the iMac instead (data is local), but the audit-log idea transfers.
- **Source adapters as one-file plugins** — each ingestion source (MTEC, SBIR, USAspending) should be one module with a common interface, exactly like OpenPitch's adapter pattern.

**What it does NOT give us:** military-medical data. Its universe is global AI startups — 140+ profiled. Zero overlap with MTEC, CDMRP, or TechLink. Its podcast/EDGAR pipeline is irrelevant to DoD funding sources.

## OpenBook (github.com/iloveitaly/openbook) — MIT, ~65 stars

**What it is:** An open-source database of venture capital *firms* and their investors, built on Dolt (git for databases). Scrapes firm websites, categorizes pages with LLMs, stores team members with contact info. Queryable via SQL. Focused on helping founders find the right VC to pitch.

**What MilMed Connect PitchBook can borrow:**

- **LLM-assisted web scraping** — converting raw HTML to simplified markdown, then extracting structured JSON with an LLM. This is a proven pattern for sites without APIs (relevant to CDMRP and TechLink scraping).
- **Dolt as a database** — git-versioned data with branching and PRs for corrections. Interesting for the entity-resolution review queue: a human could review a borderline match as a Dolt PR.
- **SQL-queryable public data** — the idea that the dataset itself should be open and queryable, not locked behind an app.

**What it does NOT give us:** company or funding data. It's a directory of VC *firms*, not of startups or their awards. Zero overlap with our sources.

## The verdict: borrow patterns, don't integrate code

Neither repo contains data or code that maps to MTEC, SBIR, CDMRP, USAspending, or TechLink. Forking them would mean deleting most of what they are. The right move is **pattern adoption**:

1. **Confidence + provenance on every score** (from OpenPitch) — add `confidence` and `sources` fields to the scores table.
2. **One-file source adapters** (from OpenPitch) — each ingest module implements a common interface.
3. **LLM-assisted scraping for no-API sources** (from OpenBook) — use for CDMRP and TechLink.
4. **Contradiction flags** (from OpenPitch) — already in the LLM scoring prompt; keep it.

## Optional: run OpenPitch alongside as a complement

One legitimate option: add OpenPitch's MCP server as a *second* MCP server on the iMac, alongside MilMed Connect PitchBook's. Eric's agent could then query both — MilMed Connect PitchBook for dual-use medtech discovery, OpenPitch for the broader AI startup landscape. They don't conflict; they cover different universes. This is a one-line MCP config addition, not a code integration.

## What to avoid

- Do not fork OpenPitch or OpenBook into MilMed Connect PitchBook. Different data, different domain, different architecture (OpenPitch is GitHub-Actions-centric; we are iMac-local).
- Do not copy their data files. OpenPitch's committed JSON is AI-startup claims; importing it would pollute the medtech database.
- Do not adopt Dolt as the primary database. Postgres 16 is already the stack; Dolt adds operational complexity for no gain at our scale (13.5k orgs, not millions).