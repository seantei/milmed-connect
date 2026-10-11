# MILMED Connect

**A PitchBook-style intelligence platform for dual-use medical technology — built entirely on public DoD sources.**

MILMED Connect aggregates, scores, and surfaces dual-use medtech companies across the fragmented landscape of military medical funding: MTEC, SBIR/STTR, CDMRP, USAspending, TechLink, DIU, and more. Think PitchBook, but for the military medical innovation ecosystem — free, open-source, self-hostable, and MCP-native so AI agents can query it directly.

## Why this exists

- **PitchBook costs $20k+/year** and its military-medical coverage is thin and stale.
- **The data is public but scattered.** MTEC's directory alone has 13,506 organizations. CDMRP has funded 22,456 awards totaling ~$19.7B. TechLink lists ~7,000 DoD/VA lab technologies (1,945 medical). No one has unified them.
- **Dual-use discovery is a needle-in-a-haystack problem.** A company doing AI triage or wearable sensors might appear in three sources and be invisible in the rest.
- **The moat is scoring + relationships, not scraping.** Anyone can build the list. The value is in rating dual-use fit and connecting companies to the right DoD investigators and PIs.

## Architecture

```
├── ingest/          # Source-specific crawlers & API clients
│   ├── mtec.py        # MTEC member directory (13.5k profiles, paginated, filterable)
│   ├── sbir.py        # SBIR.gov API + bulk downloads
│   ├── usaspending.py  # USAspending.gov bulk download API
│   └── cdmrp.py        # CDMRP funding history (scraped; no public API)
├── normalize/       # Entity resolution & schema unification
│   ├── schema.sql      # Unified company/award/technology schema
│   ├── resolve.py      # Splink-based probabilistic matching across sources
│   └── clean.py        # cleanco + custom medtech name normalization
├── score/           # Dual-use scoring engine
│   ├── rubric.yaml     # MTEC focus-area weights + capability tags
│   ├── llm_score.py    # LLM-based dual-use fit scoring with confidence
│   └── signals.py      # Civilian traction, FDA status, seeking-investment flags
├── api/             # FastAPI service
├── mcp/             # MCP server for AI agent access
└── dashboard/       # React search UI (company profiles, funding maps, alerts)
```

## Data sources (ingestion priority)

| Priority | Source | Records | Access | Notes |
|----------|--------|---------|--------|-------|
| 1 | MTEC directory | 13,506 orgs | Public HTML, paginated (`?page=N`), filterable | Profiles at `/life-sciences/{slug}`: description, products, expertise, key technologies, email, address, website, news |
| 2 | SBIR.gov | All SBIR/STTR awards since inception | Official JSON API + bulk XLS/JSON/XML downloads | Filter by agency (DoD), keywords, NAICS |
| 3 | USAspending.gov | DoD medical contracts | Official bulk-download API (`/api/v2/download/`) | Filter by CFDA 12.420 (military medical research) |
| 4 | CDMRP | 22,456 awards, $19.7B | Scraped funding-history pages | No public award database; Dimensions as fallback |
| 5 | TechLink | ~1,945 medical technologies | Keyword scrape of marketplace | No API; 100+ labs; medical is largest category |
| 6 | DIU Digital OnRamp | Growing | MVP/signup | Still early; future source |
| 7 | Grants.gov | Medical CFDA awards | Public API | Cross-check with USAspending |

## Dual-use scoring rubric

### Tier 1 — Focus area fit (weight 40%)
MTEC's six focus areas, weighted by award volume:
- Combat Casualty Care — 27% of MTEC awards ($368M). Point-of-injury, en-route, facility care; hemorrhage, trauma, evacuation.
- Military Operational Medicine — 22% ($260M). Environmental stressors, physical/psychological injury prevention, human performance.
- Medical Simulation & Information Sciences — 20% ($206M). Training, telemedicine, decision support, EHRs.
- Military Infectious Diseases — 14% ($196M). Point-of-care diagnostics, vaccines, antimicrobials, wound infection, sepsis.
- Clinical & Rehabilitative Medicine — 12% ($162M). Reconstruction, rehab, prosthetics, vision.
- Chemical & Biological Threat Reduction — 1% ($15M). Prophylaxes, treatments, detection.

### Tier 2 — Capability tags (weight 25%)
MTEC's structured taxonomy: Diagnostics, Biologics, Drugs, Medical Devices, Digital Health Technologies, Routes of Administration. Plus expertise areas: sepsis management, hemorrhage, autonomous care, robotics/evacuation, psychological health, musculoskeletal injury.

### Tier 3 — Dual-use signal (weight 20%)
- Civilian market traction (revenue, customers, commercial product)
- FDA clearance/approval status
- Clinical trial activity (ClinicalTrials.gov)
- Peer-reviewed publications

### Tier 4 — Readiness (weight 15%)
- Clinical/regulatory stage (TRL)
- Team background (military, clinical, regulatory)
- Currently seeking investment (MTEC flags 163 orgs)
- Company size, location, funding history

## Entity resolution strategy

1. **Normalize** every company name with `cleanco` (strip Inc/LLC/Corp) + custom rules ("149 Medical" vs "149Medical").
2. **Block** candidates by: normalized name prefix, state, NAICS/CFDA, and shared award numbers.
3. **Score** pairs with Splink (Fellegi-Sunter probabilistic) using: name similarity (Jaro-Winkler + token sort), website domain, address, personnel overlap, award co-occurrence.
4. **Cluster** with connected components; manual review queue for borderline matches (0.7–0.9).
5. **LLM assist** for ambiguous cases: "Are 'NIRSense' and 'NIR Sense Inc' the same company given these profiles?"

Open-source libraries to leverage:
- **Splink** (moj-analytical-services) — probabilistic record linkage, SQL/Spark backends
- **dedupe** (dedupeio) — active-learning dedup
- **cleanco** — company name cleaning
- **goldenmatch** (benseverndev-oss) — zero-config ER with MCP tools, 250M rows
- **eridu** (Graphlet-AI) — deep fuzzy name matching via sentence transformers
- **EMM** (ing-bank) — scalable company-name matching at millions scale

## LLM scoring prompt (draft)

```
You are a dual-use medical technology analyst. Given a company profile
(description, products, expertise areas, key technologies, funding history),
rate its fit for MTEC's focus areas on a 0-100 scale.

Focus areas and weights:
- Combat Casualty Care (27%): point-of-injury trauma, hemorrhage control,
  en-route care, evacuation, prolonged field care
- Military Operational Medicine (22%): environmental health, human
  performance, psychological injury, musculoskeletal
- Medical Simulation & Info Sciences (20%): telemedicine, training,
  decision support, remote monitoring
- Military Infectious Diseases (14%): point-of-care diagnostics,
  antimicrobials, vaccines, wound infection, sepsis
- Clinical & Rehabilitative Medicine (12%): reconstruction, rehab,
  prosthetics, vision restoration
- Chemical & Biological Threat Reduction (1%): detection, prophylaxis

Also rate: civilian dual-use potential (0-100), regulatory readiness
(0-100), and team strength (0-100). Cite specific evidence from the
profile for each score. Flag contradictions (e.g., claims FDA clearance
but no 510(k) found).

Return JSON: {focus_area_scores: {...}, dual_use: N, regulatory: N,
team: N, evidence: [...], flags: [...]}
```

## Build phases

### Phase 1 — MVP (weeks 1–4)
- [ ] MTEC crawler: all 13.5k profiles → Postgres
- [ ] SBIR.gov DoD medical awards → Postgres
- [ ] Basic schema + Splink dedup
- [ ] Rule-based scoring (no LLM yet) — keyword + tag matching
- [ ] Simple search API (FastAPI)

### Phase 2 — Intelligence (weeks 5–8)
- [ ] LLM scoring layer with confidence + evidence citations
- [ ] USAspending + CDMRP ingestion
- [ ] Entity resolution across all sources
- [ ] Company profile pages with funding timeline

### Phase 3 — Platform (weeks 9–12)
- [ ] React dashboard: search, filters, funding maps, alerts
- [ ] MCP server for AI agent queries
- [ ] Daily refresh pipeline (GitHub Actions)
- [ ] Email/Slack alerts for new matches

### Phase 4 — Differentiation
- [ ] TechLink technology-to-company matching
- [ ] DIU Digital OnRamp integration
- [ ] Relationship graph: companies ↔ DoD PIs ↔ labs ↔ programs
- [ ] Proposal-language generator for MTEC solicitations

## Tech stack

- **Backend:** Python 3.12, FastAPI, Postgres 16, Redis
- **Ingestion:** httpx + BeautifulSoup4 (MTEC), official APIs (SBIR, USAspending)
- **ER:** Splink + cleanco + eridu embeddings
- **Scoring:** LLM (Claude/GPT) with structured output + rubric YAML
- **Frontend:** React 18 + Vite, Tailwind, Recharts
- **MCP:** FastMCP server exposing search/score/alert tools
- **Infra:** Local-first — Postgres and Redis installed directly on the machine (no Docker, no containers). GitHub Actions for daily refresh runs.

## Legal / ethics notes

- All source data is public domain or publicly accessible.
- MTEC profiles include contact emails — respect opt-out; add a suppression list.
- No classified or CUI data. Everything here is unclassified public info.
- Rate-limit crawlers; identify the bot in User-Agent.

## License

MIT. Data sourced from public U.S. government sources.

---

*Built to show MILMED Connect (Beaverton, OR) what a unified dual-use medtech discovery layer could look like.*