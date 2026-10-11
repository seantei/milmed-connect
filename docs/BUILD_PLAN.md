# MilMed Connect PitchBook — Revised Build Plan

**12-week commitment: a validated core product that answers three approved demo queries with cited evidence.** TechLink, the relationship graph, and the proposal-language generator are a separately estimated extension after the core proves useful.

No Docker. Everything runs locally on the MILMED Connect iMac. Postgres 16 and Redis installed directly via Homebrew. API keys are optional for the core; AI scoring is an accelerator. Local-first with optional hosted AI — what leaves the machine is explicit and limited.

## Phase 0 — Foundations (week 1)

**Goal:** Prove source access, define the schema and evidence model, agree on scoring with Eric and Kirtly, and create labeled test cases. API keys optional.

- [ ] Verify source access and document limitations:
  - MTEC directory: ~13,499 organizations (not 13,506 members). Homepage advertises 800+ members. Directory entries ≠ members. Store membership as a separately sourced status.
  - SBIR.gov API is under maintenance. Use official downloadable files (XLS/JSON/XML) as primary; API as fallback when available.
  - USAspending: CFDA/Assistance Listing applies only to financial assistance (grants), not contracts. Split grants and contracts into separate ingestion paths.
  - CDMRP: funding-history pages do not yield company-level award histories. Award search directs to DTIC Dimensions; public access has been unstable. Test and document usable fields before promising the second demo query.
  - FDA and ClinicalTrials.gov: required for the wearables demo; add as sources or narrow that demo.
- [ ] Check iMac hardware and supported software versions (Postgres 16, Redis, Python 3.12, Node 20+).
- [ ] Define the evidence model: every factual claim links to its source record, retrieval date, and supporting evidence. Preserve original records and versioned scores.
- [ ] Define the scoring rubric with Eric and Kirtly: focus-area weights (normalize to 100%), dual-use signal, readiness. Missing evidence = "unknown," not low quality or contradiction. One output schema across rules, Jev, and LLMs. Composite calculated deterministically in code using the rubric's 40% evidence-weight minimum.
- [ ] Create labeled test cases with Eric and Kirtly: three approved demo questions with manually verified expected answers. Include an ambiguous company match and an honest "insufficient evidence" result.
- [ ] Create the 50-company labeled set and 200-pair matching set for later calibration. Human approval required — do not let the bot fabricate labels.
- [ ] Record approved outputs in named files (see docs/INSTRUCTIONS.md). These are mandatory inputs to later phases.

**Done when:** Source limitations documented, schema and evidence model defined, scoring agreed, demo questions approved with verified answers, labeled sets created and saved in named files.

## Phase 1 — Thin end-to-end (weeks 2–4)

**Goal:** Two sources, conservative company matching, rule-based scoring, simple search. Show Eric and Kirtly a useful result here.

- [ ] Install Postgres 16 and Redis directly via Homebrew. Create the `milmed` database and apply `normalize/schema.sql`.
- [ ] MTEC crawler: all directory profiles into Postgres, dated snapshot, checkpointed and resumable. Rate-limit 1 req/2s, identify bot in User-Agent. Store membership status separately.
- [ ] SBIR.gov DoD medical awards via official downloadable files (primary). API if available.
- [ ] Conservative entity resolution: high auto-merge precision target with held-out evaluation set including difficult negative examples. Support undoing merges. Threshold agreed in Phase 0.
- [ ] Rule-based scoring from tags and keywords. Missing evidence = "unknown."
- [ ] Basic FastAPI: `/companies`, `/companies/{id}`, `/search`. Simple React search page.
- [ ] Evidence links on every displayed claim: source record, retrieval date, supporting evidence.
- [ ] Reliability: replaying ingestion creates no duplicates; interrupted runs resume; failed sources preserve the last good snapshot; reviewed merges survive refreshes; backup restoration demonstrated.
- [ ] API key discovery wired as an explicit credential contract (see DEPLOYMENT.md). Optional for this phase.

**Done when:** Two sources ingested, search returns results with evidence links, conservative matching passes the held-out set, backup restoration demonstrated, Eric and Kirtly see a useful result.

## Phase 2 — Improve against feedback (weeks 5–8)

**Goal:** Improve matching and ranking against Eric and Kirtly's feedback. Add the sources required by the chosen demo questions. Rules-only path must pass all core tests. AI scoring is optional.

- [ ] Incorporate feedback from Phase 1 demo.
- [ ] Add sources required by the approved demo questions (USAspending grants and contracts as separate paths, CDMRP if access is viable, FDA/ClinicalTrials.gov if the wearables demo is kept).
- [ ] Deduplicate awards across sources before adding funding totals.
- [ ] Rules-only path passes all core tests against the labeled set.
- [ ] If an approved usable AI provider is enabled (Jev or LLM), benchmark it against the labeled set. Jev uses `TYPESAFE_API_KEY`. Otherwise mark the AI layer explicitly disabled. Do not add a second AI layer until the first is calibrated.
- [ ] Cross-source matching with the same high-precision target and undo support.
- [ ] Company profile pages with funding timelines and evidence links.

**Done when:** Rules-only path passes all core tests. If an approved usable AI provider is enabled, its evaluation also meets the agreed criteria; otherwise it is explicitly marked disabled. Demo questions answer with cited evidence. No double-counting of awards.

## Phase 3 — Platform and formal demo (weeks 9–12)

**Goal:** MCP, reliable refreshes, alerts, operational testing, and the formal demo.

- [ ] MCP server (FastMCP, read-only for the demo): `search_companies`, `score_company`, `get_funding_history`, `find_dual_use_matches`, `get_new_matches`, `get_solicitation_alerts`.
- [ ] React dashboard: search, profiles, funding map, alerts, review queue.
- [ ] Daily refresh via cron on the iMac with refresh budgets, overlap prevention, missed-run recovery, and log rotation. Behavior defined for when the iMac sleeps or restarts.
- [ ] Email/Slack alerts for high-score companies. Suppression list respected.
- [ ] Performance: dashboard and MCP return the same correct results and citations. Measure a proposed p95 target under five seconds on the actual iMac with a declared dataset and repeated runs. Precompute scores.
- [ ] Bot prepares the formal demo (three approved queries, including an ambiguous match and an honest "insufficient evidence" result) and opens a PR. Eric and Kirtly approve usefulness — the bot does not declare the meeting delivered on their behalf.

**Done when:** Bot has prepared the demo and opened the PR. Eric and Kirtly have reviewed and approved usefulness separately.

## Phase 4 — Extension (separately estimated)

TechLink ingestion with company matching, relationship graph, proposal-language generator. Start only after the core product proves useful to Eric and Kirtly.

## Demo queries (to be approved in Phase 0)

1. Companies with point-of-care diagnostics + hemorrhage control that have SBIR awards but no MTEC membership (membership status must be separately sourced).
2. CDMRP-funded companies currently seeking investment (depends on viable CDMRP access).
3. Dual-use wearables: operational medicine tags + civilian FDA clearance + active clinical trials (requires FDA and ClinicalTrials.gov sources, or narrow the demo).

## Position

"A focused military-medical intelligence platform that can replace specific research workflows." Not a complete PitchBook replacement, and not a fixed annual savings claim until the core is validated.
