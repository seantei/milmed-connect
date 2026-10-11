# MilMed Connect PitchBook — Build Plan (add-on to AetherForge + Signal Registry)

**What this is:** an add-on to the systems already running on the MILMED Connect iMac, not a new platform.
- **AetherForge** is the data foundation: Postgres 17 + pgvector, ~21,800 companies, ~102,000 awards, ~81,000 FDA records, ~181,000 patents, ~41,000 SEC filings, provenance on every fact, rules-based entity resolution, FastAPI + Next.js "hub" UI.
- **Signal Registry** is Helix's deep layer: in-depth company profiles, the DU-T2 assessments, topic analyses, the solicitation watcher, and the ops dashboard.

**What the add-on adds:** (1) the MTEC organization directory, its capability tags, membership status and "seeking investment" flag, matched to AetherForge companies; (2) a rules-based dual-use *pre-screen* score across all AetherForge companies, aligned with DU-T2; (3) three approved demo queries answered with cited evidence; (4) a read-only MCP server over the existing API. Nothing is rebuilt from scratch.

**Re-estimated commitment: about 7 weeks** for the validated core (was 12), because sources, storage, entity resolution, evidence model and UI already exist. TechLink, proposal-language generation and further graph work remain a separately estimated extension.

## Hard constraints (unchanged)
- No Docker, no containers. Local on the MILMED Connect iMac. Eric and Kirtly own everything.
- Use AetherForge's existing Postgres 17 cluster (new schema `milmed_addon`). **No new Postgres 16 or Redis** unless a measured need is shown in Phase 0/1 and approved.
- Evidence model: every claim links to source record, retrieval date and supporting evidence; missing evidence = "unknown"; composite computed deterministically in code with the 40% evidence-weight minimum.
- Crawlers: 1 request / 2 seconds, bot identified in User-Agent, checkpointed and resumable.
- Respect the MTEC contact-email suppression list (stored in the DB).
- Rules-only path must always work; AI is optional and credential-gated (see DEPLOYMENT.md).
- Additive only: the add-on never edits or deletes existing AetherForge or Signal Registry rows; it writes to its own schema/tables and reads the rest.
- Human approval gates: the bot prepares drafts and PRs; Eric and Kirtly approve. Never invent approvals or labels.

## What already exists (verified read-only, 2026-10-11)
| Need | Already in place | Gap for the add-on |
|---|---|---|
| USAspending (contracts + grants split) | AetherForge, ~67,200 awards, refreshed daily | none |
| SBIR/STTR | NIH RePORTER (HHS SBIR) + USAspending contract-side; Signal Registry SBIR connector. SBIR.gov API blocked (403/429) | official SBIR bulk downloads, if Phase 0 shows a coverage gap |
| CDMRP | AetherForge W81XWH keyword lane via USAspending: only 7 CDMRP awards; no company-level CDMRP source | test DTIC/CDMRP access; demo 2 may need narrowing |
| FDA, ClinicalTrials.gov | FDA ~80,700 records in AetherForge; CT.gov in Signal Registry | link CT.gov to AetherForge companies if demo 3 kept |
| MTEC | Solicitations only (Signal Registry watcher, AetherForge solicitation feed). No directory, member, tag or MTEC-award ingest | **new: MTEC directory crawl + membership** |
| "Seeking investment" | not present anywhere | **new: from MTEC profile filter** |
| Entity resolution | AetherForge deterministic rules, strong-ID veto, 2,000-pair gold set generated from shared-UEI / name+city evidence (not hand-labeled) | **new: human-labeled MTEC-vs-existing pairs** |
| API | AetherForge FastAPI: /search, /profile, /graph, /solicitations, /analyst/screen | add endpoints for pre-screen + MTEC fields |
| MCP | none | **new: thin read-only MCP over the API** |
| Dual-use scoring | DU-T2 (Kirtly's 21-subsection framework) in Signal Registry, 7 assessments, none yet validated | **new: pre-screen aligned to DU-T2** |
| UI | AetherForge hub (port 3420), Signal Registry ops dashboard (3010) | add a Screening view; no new dashboard |

## Phase 0 — Foundations (week 1)
- [ ] `docs/phase0/EXISTING_SYSTEMS.md`: inventory of AetherForge and Signal Registry (tables, counts, sources, refresh jobs, owners, how Eric and Kirtly use each), confirmed by Eric and Kirtly.
- [ ] `SOURCE_FINDINGS.md`: MTEC directory (~13,499 orgs; directory ≠ members; membership separately sourced), SBIR bulk-download coverage vs what AetherForge already has, CDMRP access test, CT.gov linkage. Document limits honestly.
- [ ] `SCHEMA_AND_EVIDENCE.md`: new `milmed_addon` tables (mtec_orgs, mtec_membership, mtec_tags, addon_matches, prescreen_scores, suppression_list) referencing AetherForge companies and its provenance columns.
- [ ] `SCORING_AGREEMENT.md`: pre-screen rubric and its mapping to DU-T2 (TECH/TEAM/TRAN), approved by Kirtly. Pre-screen is labeled "pre-screen, not DU-T2".
- [ ] `DEMO_QUESTIONS.md`: three queries with manually verified expected answers, one ambiguous match, one honest "insufficient evidence".
- [ ] `LABELED_SETS/`: a 50-company pre-screen set, and **MTEC-vs-existing match pairs** (target ~100, hardest first) labeled by Eric/Kirtly. The existing 2,000-pair AetherForge gold set is reused for regression; it is not relabeled.
- [ ] `HARDWARE.md`: uses existing Postgres 17 (port 55432), Python venv and Node already on the iMac; disk headroom; backup job coverage. No new Postgres 16/Redis.
- [ ] `APPROVALS.md`: Eric and Kirtly sign-off lines, empty until they sign.

**Stop rule:** before every later phase, re-read these files; if any artifact or approval is missing, stop and report.

## Phase 1 — MTEC into AetherForge (weeks 2–3)
- [ ] MTEC directory crawler (1 req/2s, UA, checkpoints, dated snapshots) into `milmed_addon`; membership status stored separately; suppression list enforced.
- [ ] Match MTEC orgs to AetherForge companies using AetherForge's ER rules; ambiguous pairs to a review queue with undo. Must pass the labeled MTEC pairs and not regress the existing gold set.
- [ ] Read-only API endpoints for MTEC fields; replay creates no duplicates; backup restore of the add-on schema demonstrated.
- **Done when:** MTEC ingested and matched, held-out pairs pass, evidence links shown, Eric and Kirtly see a useful result. Open a PR and stop.

## Phase 2 — Pre-screen + demo queries (weeks 4–5)
- [ ] Rules-only pre-screen over all matched companies, written to `prescreen_scores` with evidence ids; unknown-aware.
- [ ] Fill source gaps the approved demos need (SBIR bulk files, CDMRP if viable, CT.gov linkage), deduplicated against existing awards (no double-counting).
- [ ] Optional AI (Jev/LLM) evaluated only if an approved usable provider is enabled; else marked disabled.
- [ ] Shortlist hand-off: top pre-screen companies can be sent to Helix for a full DU-T2 in Signal Registry (human-triggered).
- **Done when:** rules-only path passes the labeled set; demos answer with citations. Open a PR and stop.

## Phase 3 — Surface + formal demo (weeks 6–7)
- [ ] Screening view inside the existing AetherForge hub (no new dashboard).
- [ ] Read-only MCP server (`search_companies`, `score_company`, `get_funding_history`, `find_dual_use_matches`, `get_new_matches`, `get_solicitation_alerts`) over the same API; same results and citations as the UI.
- [ ] Add-on refresh added to the existing launchd schedule, with overlap prevention and missed-run recovery; alerts respect the suppression list.
- [ ] Prepare the formal demo and open a PR. Eric and Kirtly approve usefulness separately.

## Phase 4 — Extension (separately estimated)
TechLink, deeper relationship graph, proposal-language generator. Only after the core proves useful.

## Demo queries (to be approved in Phase 0)
1. Point-of-care diagnostics + hemorrhage control companies with SBIR awards but no MTEC membership.
2. CDMRP-funded companies currently seeking investment (only 7 CDMRP awards exist today; depends on Phase 0 CDMRP access test).
3. Dual-use wearables: operational-medicine tags + FDA clearance + active clinical trials.

## Position
"An add-on to MILMED's AetherForge and Signal Registry that brings in MTEC and a quick dual-use pre-screen, so you can ask focused military-medical questions across thousands of companies — with DU-T2 still the deep assessment." Not a PitchBook replacement; no savings claim until validated.
