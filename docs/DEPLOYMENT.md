# MILMED Connect — Deployment & API Keys

## Target machine: MILMED Connect iMac

This project is designed to run **locally on the MILMED Connect iMac** (macOS). No cloud hosting, no Docker, no containers. Everything — database, API, dashboard, MCP server — lives on that one machine so Eric and Kirtly own the data and the pipeline outright.

### Prerequisites (install directly, no containers)

- **macOS** with Homebrew
- **Postgres 16**: `brew install postgresql@16 && brew services start postgresql@16`
- **Redis**: `brew install redis && brew services start redis`
- **Python 3.12**: `brew install python@3.12` (use a venv: `python3.12 -m venv .venv && source .venv/bin/activate`)
- **Node.js 20+**: `brew install node` (for the React dashboard and npm tooling)

### First-time setup

1. Clone the repo: `git clone https://github.com/seantei/milmed-connect.git`
2. Create the database: `createdb milmed`
3. Apply schema: `psql milmed < normalize/schema.sql`
4. Copy env template: `cp .env.example .env`
5. Run the crawler: `python -m ingest.mtec` (checkpoints progress; safe to re-run)
6. Start the API: `uvicorn api.main:app --reload`
7. Start the dashboard: `cd dashboard && npm install && npm run dev`

### Daily operation

- Nightly refresh via **cron** (not GitHub Actions — the data lives on the iMac):
  `0 2 * * * cd /path/to/milmed-connect && .venv/bin/python -m ingest.run_all >> logs/ingest.log 2>&1`
- Dashboard available at `http://localhost:5173` (dev) or served by the FastAPI static mount in production.
- MCP server runs alongside the API for AI-agent queries.

## API key handling

**Use whatever API keys are already stored on the MILMED iMac.** Do not create new keys, do not hardcode secrets, do not commit anything to `.env`.

### Key discovery order (first match wins)

1. **Project `.env` file** (gitignored) — if present, load it.
2. **Shell profile** — `~/.zshrc` / `~/.zprofile` exports (e.g., `export ANTHROPIC_API_KEY=...`).
3. **macOS Keychain** — `security find-generic-password -s "<service>" -w` for services like `anthropic`, `openai`, `typesafe`.
4. **Standard CLI config files** — `~/.config/<tool>/` directories (Claude Code, Codex, etc.).

### Implementation

`api/config.py` (or equivalent) should:

- Load `.env` via `python-dotenv` if it exists.
- Fall back to `os.environ` (populated by shell profile).
- Fall back to Keychain lookup via `subprocess` calling `security find-generic-password`.
- Log *which source* supplied each key (never the key itself) so setup is debuggable.
- Fail loudly with a clear message naming the missing key and where to put it.

### Keys the project may need

| Key | Used for | Required? |
|-----|----------|-----------|
| `ANTHROPIC_API_KEY` or `OPENAI_API_KEY` | LLM scoring layer (evidence citations, contradiction flags) | Phase 2+ |
| `JEV_API_KEY` | Jev decision model (fast rubric scoring, ER borderline queue) | Phase 2+ (optional — rule-based scoring works without it) |
| None | MTEC, SBIR.gov, USAspending, CDMRP, TechLink, Grants.gov | Public sources — no key needed |

### Security rules

- `.env` is in `.gitignore`. Never commit it.
- Never print or log key values.
- The suppression list for MTEC contact emails lives in the DB, not in env.
- All ingested data is unclassified public information.

## Jev integration (Phase 2)

Jev (TypeSafe AI's System One decision model) handles the high-volume, low-reasoning decisions. The LLM handles the low-volume, high-reasoning decisions. Jev decides; the LLM explains; code acts.

### Where Jev earns its keep

- **Rubric scoring at scale**: Jev `score` questions for each MTEC focus area (combat casualty care, operational medicine, etc.) across 13.5k+ profiles. 70–500 ms per call, $0.042/million input tokens, output free. Batch 7–10 questions per request.
- **Entity-resolution borderline queue**: Jev `noul` questions — "are these two records the same company?" — returning a probability. Only 0.7–0.9 matches reach human review.
- **Contradiction flags**: Jev `noul` — "does this profile claim FDA clearance without a 510(k) in evidence?"
- **Routing**: Jev `choice` — which ingestion source or scoring tier handles this record.

### Where the LLM stays

- Evidence citations and plain-English summaries (reasoning required).
- Complex multi-hop queries through the MCP server.
- Anything Jev's independent benchmarks show it trails on (rating answer quality, open-ended reasoning).

### Fallback

If no `JEV_API_KEY` is found, the scoring engine degrades gracefully to rule-based keyword/tag scoring. The platform still works — Jev is an accelerator, not a dependency.

### Official resources

- TypeSafe org: github.com/typesafe-ai (SDKs, agent skill)
- Docs: docs.typesafe.ai
- Python SDK: `typesafe-sdk-python`
- Agent skill (Claude Code / Codex / Cursor): `npx skills add typesafe-ai/skills`

## Value for MILMED Connect

- **Replaces $20k+/year PitchBook** for military-medical coverage with a free, owned, local system.
- **Answers in seconds** the cross-source questions Eric and Kirtly currently research manually across 5+ sites.
- **Data stays on the iMac** — no third-party hosting of company contacts or funding intelligence.
- **MCP-native** — any AI agent (including Grok) can query it directly.
- **MIT-licensed**, fully auditable, no vendor lock-in.