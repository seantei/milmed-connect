# MilMed Connect PitchBook — Deployment & Credentials

## Target machine: MILMED Connect iMac

This project is an **add-on to AetherForge and Signal Registry**, which already run **locally on the MILMED Connect iMac** (macOS). No cloud hosting, no Docker, no containers. The add-on lives on that same machine and reuses AetherForge's database, API and hub UI. Eric and Kirtly own the data and the pipeline outright.

**Local-first with optional hosted AI.** The database stays local. Jev, Claude, or GPT scoring sends selected content to hosted providers. What leaves the machine is explicit and limited. Spending limits should be set on any hosted AI account.

### Prerequisites (already installed on the iMac — reuse, do not reinstall)

- **Postgres 17 + pgvector**: AetherForge's existing cluster (launchd `com.milmedconnect.aetherforge-db`, port 55432). The add-on uses a new schema `milmed_addon` in the `aetherforge` database.
- **No new Postgres 16 and no Redis** unless Phase 0/1 measures a real need and Eric/Kirtly approve it. Checkpoints live in Postgres tables.
- **Python**: AetherForge's existing venv (`~/aetherforge/.venv`); **Node**: already present for the hub UIs.

### First-time setup (Phase 1, after Phase 0 approval)

1. Snapshot the AetherForge database (existing backup job) before any migration.
2. Apply the additive migration that creates `milmed_addon` (no changes to existing tables).
3. Copy env template: `cp .env.example .env` (gitignored).
4. Run the MTEC crawler (1 req/2s, identified User-Agent, checkpointed, resumable, suppression list enforced).
5. Extend the existing AetherForge API with read-only add-on endpoints; add the Screening view to the existing hub UI.

### Daily operation

- Add-on refresh scheduled with **launchd** alongside the existing AetherForge jobs, with refresh budgets, overlap prevention, missed-run recovery and log rotation. Defined behavior when the iMac sleeps or restarts.
- Results appear in the existing AetherForge hub (port 3420); deep DU-T2 assessments stay in Signal Registry via Helix.
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

- An add-on to AetherForge and Signal Registry that brings in MTEC and a dual-use pre-screen.
- Answers cross-source questions in seconds that Eric and Kirtly currently research manually.
- Data stays on the iMac. Optional hosted AI is explicit and limited.
- MCP-native. MIT-licensed. No vendor lock-in.