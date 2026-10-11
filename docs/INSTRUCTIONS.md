# MilMed Connect PitchBook — Revised Step-by-Step Instructions

Everything you need, in order. Each step says **when** to do it, **where** to do it, and **exactly what to paste**. Phase 0 artifacts are mandatory inputs to every later phase.

**Approved Phase 0 artifact files (must exist before Phase 1):**
- `docs/phase0/SOURCE_FINDINGS.md`
- `docs/phase0/SCHEMA_AND_EVIDENCE.md`
- `docs/phase0/SCORING_AGREEMENT.md`
- `docs/phase0/DEMO_QUESTIONS.md` (with expected answers)
- `docs/phase0/LABELED_SETS/` (50-company and 200-pair sets)
- `docs/phase0/HARDWARE.md`
- `docs/phase0/APPROVALS.md` (human sign-off)

---

## STEP 0 — Foundations (on the iMac and with Eric and Kirtly, before any bot session)

**When:** Before starting any bot session. Week 1.

**Where:** Terminal on the MILMED Connect iMac, plus a conversation with Eric and Kirtly.

**What to do:**

1. Clone the repo:
```bash
cd ~
git clone https://github.com/seantei/milmed-connect.git
cd milmed-connect
```

2. Check iMac hardware and software versions (Postgres 16, Redis, Python 3.12, Node 20+). Record in `docs/phase0/HARDWARE.md`.

3. Verify source access and document limitations in `docs/phase0/SOURCE_FINDINGS.md`.

4. Agree on scoring with Eric and Kirtly. Record in `docs/phase0/SCORING_AGREEMENT.md`. Normalize focus weights to 100%. Missing evidence = "unknown." Composite calculated in code with 40% evidence-weight minimum.

5. Approve three demo questions with manually verified expected answers. Include an ambiguous company match and an honest "insufficient evidence" result. Record in `docs/phase0/DEMO_QUESTIONS.md`.

6. Create the 50-company labeled set and 200-pair matching set. Human approval required. Save under `docs/phase0/LABELED_SETS/`.

7. Record human approval in `docs/phase0/APPROVALS.md`.

8. Optional: run the key-discovery check from docs/API_KEY_CHECK.md.

**Done when:** All Phase 0 artifact files exist and human approval is recorded.

---

## STEP 1 — Phase 1: thin end-to-end (fresh Grok bot session)

**When:** After Step 0 artifacts are approved.

**Where:** Open a new Grok bot session. Paste this as the opening prompt:

```
Build Phase 1 of MilMed Connect PitchBook from the repo at github.com/seantei/milmed-connect. This runs locally on the MILMED Connect iMac — no Docker, no containers. Postgres 16 and Redis install directly via Homebrew, Python 3.12 in a venv, Node 20+ for the frontend. Follow the README architecture exactly: ingest, normalize, score, api, mcp, dashboard folders.

Read the approved Phase 0 artifacts first. Required files: docs/phase0/SOURCE_FINDINGS.md, docs/phase0/SCHEMA_AND_EVIDENCE.md, docs/phase0/SCORING_AGREEMENT.md, docs/phase0/DEMO_QUESTIONS.md, docs/phase0/LABELED_SETS/, docs/phase0/HARDWARE.md, docs/phase0/APPROVALS.md. If any required artifact or human approval is missing, report the missing items and stop. Do not invent approvals or labels.

Thin end-to-end only. Two sources: MTEC directory (all profiles, dated snapshot, checkpointed, resumable, rate-limit 1 req/2s, identify bot in User-Agent) and SBIR.gov DoD medical awards via official downloadable files (primary; API if available). Store MTEC membership as a separately sourced status — directory entries are not members.

Conservative entity resolution: high auto-merge precision target with held-out evaluation set including difficult negative examples. Support undoing merges. Use the threshold agreed in Phase 0.

Rule-based scoring from tags and keywords only. Missing evidence = "unknown," not low quality or contradiction. One output schema. Evidence links on every displayed claim: source record, retrieval date, supporting evidence. Composite calculated in code using the rubric's 40% evidence-weight minimum.

Basic FastAPI with /companies, /companies/{id}, /search. Simple React search page.

Reliability requirements: replaying ingestion creates no duplicates; interrupted runs resume; failed sources preserve the last good snapshot; reviewed merges survive refreshes; demonstrate backup restoration.

API key discovery: explicit credential contract — named provider, approved account, exact env var or project-specific Keychain item. Separate configured/authenticated/usable. Optional for this phase. Do not recursively scan unrelated CLI config or execute shell profiles. Missing optional credentials disable that feature; core continues.

When Phase 1 runs end to end and passes the reliability gates, open a pull request and stop. Eric and Kirtly review the result separately.
```

**Done when:** Two sources ingested, search returns results with evidence links, conservative matching passes the held-out set, backup restoration demonstrated. Show Eric and Kirtly a useful result.

**Then:** Review the PR, merge it, and only then start Step 2.

---

## STEP 2 — Phase 2: improve against feedback (new Grok bot session)

**When:** After Phase 1's PR merges and Eric and Kirtly have given feedback.

**Where:** Open a new Grok bot session. Paste this as the opening prompt:

```
Build Phase 2 of MilMed Connect PitchBook from the repo at github.com/seantei/milmed-connect, continuing from the merged Phase 1 branch. No Docker. Local on the iMac. Incorporate feedback from the Phase 1 demo with Eric and Kirtly.

Read the approved Phase 0 artifacts first. Required files: docs/phase0/SOURCE_FINDINGS.md, docs/phase0/SCHEMA_AND_EVIDENCE.md, docs/phase0/SCORING_AGREEMENT.md, docs/phase0/DEMO_QUESTIONS.md, docs/phase0/LABELED_SETS/, docs/phase0/HARDWARE.md, docs/phase0/APPROVALS.md. If any required artifact or human approval is missing, report the missing items and stop. Do not invent approvals or labels.

Add the sources required by the approved demo questions: USAspending grants and contracts as separate ingestion paths (CFDA applies only to grants); CDMRP only if access is viable; FDA and ClinicalTrials.gov if the wearables demo is kept. Deduplicate awards across sources before adding funding totals.

Improve matching against their feedback. Same high-precision target and undo support as Phase 1.

Rules-only path must pass all core tests. AI scoring is optional. If an approved usable AI provider is enabled, benchmark it against the labeled set. Jev uses TYPESAFE_API_KEY. Otherwise mark the AI layer explicitly disabled. Do not add a second AI layer until the first is calibrated. Missing evidence stays "unknown." No contradiction flag without affirmative conflicting evidence. Composite calculated in code using the 40% evidence-weight minimum.

Company profile pages with funding timelines and evidence links.

When Phase 2 is complete (rules-only passes; AI evaluated only if enabled), open a pull request and stop.
```

**Done when:** Rules-only path passes all core tests. If an approved usable AI provider is enabled, its evaluation also meets the agreed criteria; otherwise it is explicitly marked disabled. Demo questions answer with cited evidence. No double-counting of awards.

**Then:** Review the PR, merge it, and only then start Step 3.

---

## STEP 3 — Phase 3: platform and formal demo (new Grok bot session)

**When:** After Phase 2's PR merges.

**Where:** Open a new Grok bot session. Paste this as the opening prompt:

```
Build Phase 3 of MilMed Connect PitchBook from the repo at github.com/seantei/milmed-connect, continuing from the merged Phase 2 branch. No Docker. Local on the iMac.

Read the approved Phase 0 artifacts first. Required files: docs/phase0/SOURCE_FINDINGS.md, docs/phase0/SCHEMA_AND_EVIDENCE.md, docs/phase0/SCORING_AGREEMENT.md, docs/phase0/DEMO_QUESTIONS.md, docs/phase0/LABELED_SETS/, docs/phase0/HARDWARE.md, docs/phase0/APPROVALS.md. If any required artifact or human approval is missing, report the missing items and stop. Do not invent approvals or labels.

MCP server (FastMCP, read-only for the demo) with six tools: search_companies, score_company, get_funding_history, find_dual_use_matches, get_new_matches, get_solicitation_alerts. Dashboard and MCP must return the same correct results and citations.

React dashboard: search, profiles, funding map, alerts, review queue.

Daily refresh via cron on the iMac with refresh budgets, overlap prevention, missed-run recovery, and log rotation. Define behavior when the iMac sleeps or restarts.

Email/Slack alerts for high-score companies. Suppression list respected.

Performance: measure a proposed p95 target under five seconds on the actual iMac with a declared dataset and repeated runs. Precompute scores.

Prepare the formal demo materials (three approved queries, including an ambiguous match and an honest "insufficient evidence" result). Open a pull request and stop. Eric and Kirtly approve usefulness separately — do not declare the demo delivered on their behalf.
```

**Done when:** Bot has prepared the demo and opened the PR. Eric and Kirtly review and approve usefulness separately.

**Then:** Review the PR, merge it. Phase 4 is a separately estimated extension.

---

## STEP 4 — Phase 4: extension (separately estimated)

**When:** Only after the core product proves useful to Eric and Kirtly.

**Where:** New Grok bot session. Paste this:

```
Build Phase 4 of MilMed Connect PitchBook from the repo at github.com/seantei/milmed-connect, continuing from the merged Phase 3 branch. No Docker. Local on the iMac.

Read the approved Phase 0 artifacts first. Required files: docs/phase0/SOURCE_FINDINGS.md, docs/phase0/SCHEMA_AND_EVIDENCE.md, docs/phase0/SCORING_AGREEMENT.md, docs/phase0/DEMO_QUESTIONS.md, docs/phase0/LABELED_SETS/, docs/phase0/HARDWARE.md, docs/phase0/APPROVALS.md. If any required artifact or human approval is missing, report the missing items and stop. Do not invent approvals or labels.

This is an extension after the core product has proven useful. TechLink ingestion with company matching, relationship graph (NetworkX, interactive dashboard view, shortest-path query), and proposal-language generator (LLM layer if enabled, human-review disclaimer, version history).

Performance pass, CSV/PDF exports, backup cron.

When Phase 4 is complete, open a pull request and stop.
```

---

## Position for Eric and Kirtly

"A focused military-medical intelligence platform that can replace specific research workflows." Not a complete PitchBook replacement, and not a fixed annual savings claim until the core is validated.

## Notes

- Phase 0 artifact files are mandatory. Every phase prompt requires them and stops if they are missing.
- AI scoring is optional. The rules-only path must always pass.
- The bot prepares demos and opens PRs. Eric and Kirtly approve usefulness separately.
- Local-first with optional hosted AI. What leaves the machine is explicit and limited.
