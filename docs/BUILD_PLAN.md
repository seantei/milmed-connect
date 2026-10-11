# Build Plan — 12 Weeks to Demo

**No Docker. Everything runs locally on the MILMED Connect iMac: Postgres 16 and Redis installed directly via Homebrew, Python venv for dependencies, npm for the frontend. API keys are discovered from existing storage on the machine (.env → shell profile → macOS Keychain) — no new keys created.**

## Week 1–2: Foundation
- [ ] Install Postgres 16 and Redis directly (`brew install` + `brew services start` — no containers)
- [ ] SQL schema: companies, awards, technologies, scores, sources
- [ ] MTEC crawler (httpx + BS4), 13.5k profiles, nightly, checkpointed + resumable
- [ ] SBIR.gov DoD medical pull via API
- [ ] Basic FastAPI: /companies, /companies/{id}, /search
- [ ] API key discovery wired in `api/config.py`: .env → shell env → Keychain, with source logging

## Week 3–4: Normalization + Scoring v0
- [ ] cleanco + custom name normalization
- [ ] Splink dedup within MTEC (intra-source)
- [ ] Rule-based scoring from tags + keywords (no LLM, no Jev yet)
- [ ] Seed dashboard: React search page, top-50 scored companies

## Week 5–6: Jev + LLM Layer
- [ ] Jev scoring: rubric `score` questions per focus area, batched 7–10 per call
- [ ] Jev `noul` for ER borderline queue and contradiction flags
- [ ] Jev `choice` for source/tier routing
- [ ] LLM scoring for evidence citations + plain-English summaries (what Jev can't do)
- [ ] Graceful fallback: no JEV_API_KEY → rule-based scoring still works
- [ ] Confidence calibration on 50 hand-labeled companies

## Week 7–8: Cross-Source ER + More Ingestion
- [ ] USAspending + CDMRP ingestion
- [ ] Cross-source Splink (MTEC ↔ SBIR ↔ USAspending)
- [ ] Company profile pages: funding timeline, awards, technologies
- [ ] Review queue UI for borderline matches

## Week 9–10: Platform
- [ ] MCP server (FastMCP): search_companies, score_company,
  get_funding_history, find_dual_use_matches tools
- [ ] Alerts: new high-score companies, new MTEC solicitations
- [ ] Daily refresh via **cron on the iMac** (not GitHub Actions — data is local)
- [ ] Email/Slack alerts for new matches

## Week 11–12: Polish + Demo
- [ ] TechLink ingestion
- [ ] Relationship graph view (companies ↔ PIs ↔ programs)
- [ ] Demo script for MILMED Connect: 3 queries they actually run
- [ ] One-pager + architecture diagram for the pitch

## Demo queries for MILMED Connect

1. "Show me all companies with point-of-care diagnostics + hemorrhage
   control that have SBIR awards but no MTEC membership."
2. "Which CDMRP-funded companies are currently seeking investment?"
3. "Find dual-use wearables: military operational medicine tags + civilian
   FDA clearance + active clinical trials."

These three queries are things Eric and Kirtly do manually across 5+ sites.
If the platform answers them in seconds, the demo sells itself.