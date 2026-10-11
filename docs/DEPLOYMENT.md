# MilMed Connect PitchBook — Deployment & Credentials

## Target machine: MILMED Connect iMac

This project runs **locally on the MILMED Connect iMac** (macOS). No cloud hosting, no Docker, no containers. The database, API, dashboard, and MCP server live on that one machine. Eric and Kirtly own the data and the pipeline outright.

**Local-first with optional hosted AI.** The database stays local. Jev, Claude, or GPT scoring sends selected content to hosted providers. What leaves the machine is explicit and limited. Spending limits should be set on any hosted AI account.

### Prerequisites (install directly, no containers)

- **macOS** with Homebrew
- **Postgres 16**: `brew install postgresql@16 && brew services start postgresql@16`
- **Redis**: `brew install redis && brew services start redis`
- **Python 3.12**: `brew install python@3.12` (use a venv: `python3.12 -m venv .venv && source .venv/bin/activate`)
- **Node.js 20+**: `brew install node`

### First-time setup

1. Clone the repo: `git clone https://github.com/seantei/milmed-connect.git`
2. Create the database: `createdb milmed`
3. Apply schema: `psql milmed < normalize/schema.sql`
4. Copy env template: `cp .env.example .env`
5. Run the crawler: `python -m ingest.mtec` (checkpoints progress; safe to re-run)
6. Start the API: `uvicorn api.main:app --reload`
7. Start the dashboard: `cd dashboard && npm install && npm run dev`

### Daily operation

- Nightly refresh via **cron** on the iMac with refresh budgets, overlap prevention, missed-run recovery, and log rotation. Define behavior when the iMac sleeps or restarts.
- Dashboard available at `http://localhost:5173` (dev).
- MCP server runs alongside the API (read-only for the demo).

## Credential contract

**Use whatever API keys are already stored on the MILMED iMac, but only through an explicit contract.** Do not create new keys, do not hardcode secrets, do not commit anything to `.env`.

### Explicit contract (per provider)

For each provider, document:

- **Named provider** (e.g., Anthropic, OpenAI, TypeSafe/Jev)
- **Approved existing account/project** (which account, which project, who owns it)
- **Exact environment variable** or **project-specific Keychain item** (not a broad scan)
- **Redacted status**: configured / authenticated / usable

Separate the three states:

- **Configured**: the env var or Keychain item exists.
- **Authenticated**: a call to the provider succeeds.
- **Usable**: the account has billing and permissions for this project.

### Key discovery order (first match wins, explicit only)

1. **Project `.env` file** (gitignored) — if present and the var is set, use it.
2. **Shell profile exports** — only if explicitly sourced and the var is set in the current process environment. Do not execute shell profiles.
3. **Project-specific macOS Keychain item** — `security find-generic-password -s "milmed-connect-<provider>" -w`. Not a broad scan of all Keychain items.
4. **Do not recursively scan** unrelated CLI config directories. Do not assume Claude Code or Codex login credentials are general-purpose API keys.

### Implementation

`api/config.py` should:

- Load `.env` via `python-dotenv` if it exists.
- Fall back to `os.environ` (already populated by an explicitly sourced shell profile).
- Fall back to a project-specific Keychain lookup only.
- Log *which source* supplied each key (never the key itself).
- Report status as configured / authenticated / usable — not just "found."
- Fail loudly naming the missing key and where to put it.
- Missing optional credentials disable that feature; the core product continues.

### Keys the project may need

| Key | Used for | Required? | Notes |
|-----|----------|-----------|-------|
| `ANTHROPIC_API_KEY` or `OPENAI_API_KEY` | LLM scoring layer | Phase 2+ (optional) | Benchmark one provider first |
| `TYPESAFE_API_KEY` | Jev decision model | Phase 2+ (optional) | Official SDK reads this name, not `JEV_API_KEY` |
| None | MTEC, SBIR.gov, USAspending, CDMRP, TechLink, Grants.gov | Public sources | No key needed |

### Security rules

- `.env` is in `.gitignore`. Never commit it.
- Never print or log key values.
- Do not recursively scan unrelated CLI configuration.
- Do not execute shell profiles to discover keys.
- The suppression list for MTEC contact emails lives in the DB, not in env.
- All ingested data is unclassified public information.
- What leaves the machine for hosted AI scoring is explicit and limited. Set spending limits on hosted accounts.

## Jev integration (Phase 2, optional)

Jev (TypeSafe AI's System One decision model) handles high-volume, low-reasoning decisions. The LLM handles low-volume, high-reasoning ones. Jev decides; the LLM explains; code acts.

- Official SDK reads `TYPESAFE_API_KEY`. Endpoint: `POST https://api.typesafe.ai/v1/systemone`.
- Rubric scoring at scale: `score` questions per focus area, batched 7–10 per call.
- Entity-resolution borderline queue: `noul` questions returning a probability.
- Contradiction flags: only with affirmative conflicting evidence.
- Graceful fallback: no key → rule-based scoring. Jev is an accelerator, not a dependency.
- Benchmark one AI provider against the labeled set before adding a second.

## Value for MILMED Connect

- A focused military-medical intelligence platform that can replace specific research workflows.
- Answers cross-source questions in seconds that Eric and Kirtly currently research manually.
- Data stays on the iMac. Optional hosted AI is explicit and limited.
- MCP-native. MIT-licensed. No vendor lock-in.