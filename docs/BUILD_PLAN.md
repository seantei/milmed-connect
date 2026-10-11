# Build Plan — 12 Weeks to Demo

## Week 1–2: Foundation
- [ ] Docker Compose: Postgres 16 + Redis
- [ ] SQL schema: companies, awards, technologies, scores, sources
- [ ] MTEC crawler (httpx + BS4), 13.5k profiles, nightly
- [ ] SBIR.gov DoD medical pull via API
- [ ] Basic FastAPI: /companies, /companies/{id}, /search

## Week 3–4: Normalization + Scoring v0
- [ ] cleanco + custom name normalization
- [ ] Splink dedup within MTEC (intra-source)
- [ ] Rule-based scoring from tags + keywords (no LLM)
- [ ] Seed dashboard: React search page, top-50 scored companies

## Week 5–6: LLM Layer
- [ ] LLM scoring prompt (see LLM_SCORING_PROMPT.md)
- [ ] Batch scoring with caching + cost tracking
- [ ] Evidence citations + contradiction flags
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
- [ ] Daily GitHub Action pipeline

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