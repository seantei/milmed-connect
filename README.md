# MilMed Connect PitchBook

**An add-on to MILMED's AetherForge and Signal Registry: MTEC data plus a quick dual-use pre-screen, installed locally on the MILMED Connect iMac.**

MilMed Connect PitchBook aggregates and scores dual-use medtech companies across public military medical funding sources (MTEC, SBIR/STTR, USAspending, and others as validated). Local-first, no Docker, MCP-native. Eric and Kirtly own the data and the pipeline.

**Authoritative documents:**
- [docs/BUILD_PLAN.md](docs/BUILD_PLAN.md) — phases, scope, acceptance criteria
- [docs/INSTRUCTIONS.md](docs/INSTRUCTIONS.md) — step-by-step prompts to paste

Older prompts and claims have been retired. See [docs/MASTER_PROMPT.md](docs/MASTER_PROMPT.md) for a pointer.

## Positioning

"An add-on to AetherForge and Signal Registry that brings in MTEC and a quick dual-use pre-screen — with DU-T2 still the deep assessment." Not a complete PitchBook replacement, and not a fixed annual savings claim until the core is validated with Eric and Kirtly.

## Architecture (add-on modules; reuse AetherForge wherever it already does the job)

```
├── ingest/          # MTEC crawler (+ any gap sources the demos need)
├── normalize/       # MTEC-to-AetherForge matching, reusing AetherForge ER rules
├── score/           # Rules-based pre-screen aligned with DU-T2 (optional AI)
├── api/             # Read-only endpoints added alongside the AetherForge API
├── mcp/             # Read-only MCP server
└── dashboard/       # Screening view inside the existing AetherForge hub
```

See docs/BUILD_PLAN.md for current source priorities and limitations (MTEC directory ≠ members, SBIR downloads primary, USAspending grants/contracts split, CDMRP access tested before use).

## Scoring

Shared schema across rules, Jev, and LLM. Missing evidence = "unknown," never a low score or automatic contradiction. See [docs/SCORING_RUBRIC.md](docs/SCORING_RUBRIC.md) and [docs/LLM_SCORING_PROMPT.md](docs/LLM_SCORING_PROMPT.md).

AI scoring is optional. The rules-only path must pass all core tests. If an approved usable AI provider is enabled, its evaluation also meets the agreed criteria; otherwise it is explicitly marked disabled.

## Build phases

**Phase 0 (foundations)** must complete first: source verification, scoring agreement with Eric and Kirtly, approved demo questions with expected answers, labeled sets, hardware check. Approved artifacts are mandatory inputs to later phases.

Then Phase 1 (thin end-to-end), Phase 2 (improve against feedback), Phase 3 (platform and formal demo). Phase 4 (TechLink, graph, proposals) is a separately estimated extension.

Full details: [docs/BUILD_PLAN.md](docs/BUILD_PLAN.md) and [docs/INSTRUCTIONS.md](docs/INSTRUCTIONS.md).

## Deployment

Local on the MILMED Connect iMac, reusing AetherForge's existing Postgres 17 (new `milmed_addon` schema). No new Postgres 16 or Redis unless proven necessary. See [docs/EXISTING_CLONES.md](docs/EXISTING_CLONES.md) for the existing systems. See [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) and [docs/API_KEY_CHECK.md](docs/API_KEY_CHECK.md).

## License

MIT. Data sourced from public U.S. government sources. Review each source’s access and reuse terms separately.

---

*Built for MILMED Connect (Beaverton, OR). One-pager: [docs/ONE_PAGER.md](docs/ONE_PAGER.md).*
