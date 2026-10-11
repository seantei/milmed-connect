# Entity Resolution Strategy

## Problem

The same company appears under different names across sources:
- MTEC: "149 Medical"
- SBIR: "149 Medical, Inc."
- USAspending: "149MEDICAL INC"
- TechLink: licensee name variant

## Pipeline

1. **Normalize** (`normalize/clean.py`)
   - `cleanco` to strip legal suffixes
   - Lowercase, strip punctuation, collapse whitespace
   - Custom: remove "Inc", "LLC", "Corp", "Ltd", "Co"; handle "&" vs "and"
   - Preserve distinctive tokens (numbers like "149", acronyms)

2. **Block** (candidate pair generation)
   - Same state + similar name prefix (first 4 chars)
   - Shared award/contract number
   - Same website domain
   - Same NAICS code + overlapping personnel

3. **Score** (Splink Fellegi-Sunter)
   - Name: Jaro-Winkler + token-sort ratio (weight 0.4)
   - Website domain exact match (weight 0.2)
   - Address city/state (weight 0.15)
   - Award co-occurrence (weight 0.15)
   - Personnel name overlap (weight 0.1)

4. **Cluster**: connected components on pairs above the threshold agreed in Phase 0 (recorded in `docs/phase0/SCORING_AGREEMENT.md` or a dedicated matching-agreement file). Support undoing merges. Borderline pairs go to human review.

5. **Optional AI adjudication** for borderline cases only if an approved usable AI provider is enabled. Otherwise human review only.

## Libraries

- splink (moj-analytical-services/splink) — core ER engine
- cleanco — name cleaning
- eridu (Graphlet-AI/eridu) — embedding-based fuzzy matching
- goldenmatch (benseverndev-oss/goldenmatch) — zero-config, MCP-native, scales to 250M rows
- EMM (ing-bank/EntityMatchingModel) — millions-scale name matching
- dedupe (dedupeio/dedupe) — active learning for manual review

## Evaluation

- Hand-label 200 pairs as ground truth (created and approved in Phase 0)
- Track precision/recall/F1 per source pair, with sample size and uncertainty
- Auto-merge precision target agreed with Eric and Kirtly in Phase 0
- Known company pairs that must never merge are listed in the Phase 0 agreement
