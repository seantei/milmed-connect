# Existing Systems — What We Build On (and What We Don't Rebuild)

MILMED already runs two PitchBook-style systems on the iMac. This project is an **add-on** to them. Counts below are from a read-only check on 2026-10-11.

## AetherForge — the data foundation
- **What:** a public-record intelligence graph for medtech/biotech: who is winning non-dilutive money and clearing regulatory milestones, every fact tied to its source.
- **Stack:** Postgres 17 + pgvector (port 55432), Python FastAPI (`/search`, `/profile`, `/graph`, `/solicitations`, `/analyst/screen`), Next.js hub UI (port 3420), local Ollama models. No Docker.
- **Data:** ~21,800 companies; ~102,000 awards (USAspending ~67,200, NIH RePORTER ~34,900, CDMRP 7, Grants.gov 42, SAM 15); ~80,700 FDA regulatory records; ~180,700 patents; ~40,700 SEC filings; ~2,750 alerts; ~700 program officers. Refreshed daily (last ingest 2026-10-11).
- **Already solved:** provenance on every fact (source, first seen, last verified, confidence); deterministic entity resolution with a strong-ID veto and a 2,000-pair gold set (generated from shared-UEI / name-and-city evidence, not hand-labeled); USAspending de-duplication; LLM restricted to turning a question into a filter (it cannot invent awards); defense-health and solicitation lanes.
- **Not there yet:** MTEC directory, members, tags or MTEC awards; company-level CDMRP history; a "seeking investment" signal; an MCP server; a dual-use score.

## Signal Registry — Helix's deep layer
- **What:** deep, source-traced company profiles from eight public databases (USAspending, SBIR.gov, SEC EDGAR, NIH, NSF, openFDA, ClinicalTrials.gov, PatentsView) plus a separate context layer; the solicitation watcher; topic analyses; the ops dashboard (port 3010).
- **Stack:** TypeScript/Node, SQLite.
- **Data:** 40 deeply profiled companies, ~5,100 resolved facts, ~5,700 solicitations seen, 24 topic analyses, 7 DU-T2 assessments. Daily and weekly watcher runs via Helix (Telegram).
- **Used by:** Helix on Telegram for company reports, lookalikes, discovery, DU-T2 and topic analyses.

## DU-T2 — the deep assessment (Kirtly's framework)
- 21 subsections across TECH, TEAM and TRAN; geometric-mean total; tiers from Defense-Aligned to Defense-Distant; gap analysis, defense TAM and a Bridge-to-Dual-Use roadmap; delivered as Word reports.
- 7 assessments stored (4 draft, 3 unverified, none yet validated).
- **Relationship to this project:** the add-on's pre-screen is a fast first pass across thousands of companies; DU-T2 stays the authoritative deep assessment for the shortlist.

## What the add-on reuses vs adds
| Reuse as is | Add |
|---|---|
| AetherForge database, awards, FDA, patents, filings | MTEC directory, membership status, capability tags, "seeking investment" |
| AetherForge entity resolution and gold set | Human-labeled MTEC-vs-existing match pairs |
| Evidence/provenance model | Pre-screen score (rules-only, unknown-aware) aligned with DU-T2 |
| AetherForge API and hub UI | Read-only add-on endpoints, a Screening view, a read-only MCP server |
| Signal Registry DU-T2 and Helix | Shortlist hand-off to DU-T2 (human-triggered) |

## Duplication to avoid
A second company database, a second Postgres (16) or Redis, a competing "dual-use" number shown as if it were DU-T2, a third dashboard, and crawlers re-pulling APIs AetherForge already ingests.

## Outside open-source projects (unchanged verdict)
OpenPitch and OpenBook (both MIT) cover AI startups and VC firms, not military-medical data. Borrow patterns only (confidence-scored figures, one-file source adapters, contradiction flags); do not fork or import their data.
