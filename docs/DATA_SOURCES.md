# Data Sources Reference

## 1. MTEC (mtec-sc.org) — PRIORITY 1

- Directory: https://mtec-sc.org/search/members?page=N (13,506 orgs)
- Profile URL pattern: /life-sciences/{slug}
- Filters: capabilities, seeking investment, country, company size, key technologies
- Fields per profile: description, products, expertise areas, key technologies,
  industries, employee count, address, website, email, news/publications,
  member type, similar orgs
- No login required. No API. HTML is clean and parseable.
- Rate limit: 1 req/2s, identify bot in User-Agent.
- Sample profile verified: 149 Medical (Worcester, MA) — NIRDS1 device,
  DCS/NIRS technology, MGH license, 8 publications, full contact info.

## 2. SBIR.gov — PRIORITY 2

- API: https://api.sbir.gov (topics, awards, companies)
- Bulk downloads: XLS/JSON/XML, 10k records per file
- Filter: agency=DOD, CFDA 12.420, keywords (medical, trauma, diagnostic)
- Every SBIR/STTR award since program inception
- Python lib: AnirudhJM24/SBIR-grant-search (MIT)

## 3. USAspending.gov — PRIORITY 3

- API: https://api.usaspending.gov/api/v2/
- Bulk download: /api/v2/download/assistance/ and /download/contract/
- Filter: toptier_agency=Department of Defense, CFDA 12.420
- Scripts: bsweger/usaspending-scripts

## 4. CDMRP (cdmrp.health.mil) — PRIORITY 4

- No public award database. Funding history page lists programs + totals.
- 22,456 awards, $19.7B through FY24; $1.27B appropriated FY26 across 34 programs.
- Fallback: Dimensions (digital-science) award search; DTIC.
- Scrape funding-history tables per program.

## 5. TechLink (techlinkcenter.org) — PRIORITY 5

- Marketplace: keyword + category search, 7,000+ technologies, 1,945 medical
- 100+ DoD/VA labs. No API. HTML scrape.
- AI-powered search alpha expected late 2026 — opportunity to integrate early.
- Patent holiday: selected patents free to use for 2 years.

## 6. DIU Digital OnRamp (diu.mil/digitalonramp) — PRIORITY 6

- MVP as of 2025. Consolidates SAM.gov, SBIR, Vulcan SOF, others.
- Signup for test access. Future source once API available.

## 7. Grants.gov — PRIORITY 7

- Public REST API: /grantsws/rest/opportunities/search
- Cross-check medical CFDA awards against USAspending.

## Awesome lists

- makegov/awesome-procurement-data — curated federal procurement resources
- OlivierBinette/Awesome-Entity-Resolution — ER software catalog