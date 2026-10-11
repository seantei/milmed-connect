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

4. **Cluster**: connected components on pairs above threshold (0.85 auto-merge; 0.7–0.85 → review queue)

5. **LLM adjudication** for borderline: pass both profiles + context, ask match/no-match + rationale.

## Libraries

- splink (moj-analytical-services/splink) — core ER engine
- cleanco — name cleaning
- eridu (Graphlet-AI/eridu) — embedding-based fuzzy matching
- goldenmatch (benseverndev-oss/goldenmatch) — zero-config, MCP-native, scales to 250M rows
- EMM (ing-bank/EntityMatchingModel) — millions-scale name matching
- dedupe (dedupeio/dedupe) — active learning for manual review

## Evaluation

- Hand-label 200 pairs as ground truth
- Track precision/recall/F1 per source pair
- Target: >95% precision on auto-merges