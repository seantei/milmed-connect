# Dual-Use Scoring Rubric v1

Composite score = 0.40×Focus + 0.25×Capability + 0.20×DualUse + 0.15×Readiness

## Tier 1: Focus Area Fit (40%)

| Focus Area | Weight | What counts |
|------------|--------|-------------|
| Combat Casualty Care | 27% | Hemorrhage control, tourniquets, damage-control resuscitation, en-route critical care, prolonged field care, evacuation robotics, point-of-injury diagnostics, blast/burn/TBI |
| Military Operational Medicine | 22% | Environmental physiology (heat, cold, altitude), human performance, sleep, musculoskeletal injury prevention, psychological health, suicide prevention |
| Medical Simulation & Info Sciences | 20% | Telemedicine, remote monitoring, AI decision support, medical training (AR/VR/simulation), EHR interoperability, autonomous care |
| Military Infectious Diseases | 14% | Point-of-care molecular diagnostics (CRISPR, isothermal), antimicrobials/AMR, vaccines, wound infection (Pseudomonas, Klebsiella, Acinetobacter, S. aureus), sepsis countermeasures, vector-borne |
| Clinical & Rehabilitative Medicine | 12% | Limb loss/prosthetics, tissue regeneration, vision restoration, reconstructive surgery, rehab robotics |
| Chemical & Biological Threat Reduction | 1% | Detection, prophylaxis, treatment for CBRN exposures |

Each focus area scored 0–100 by LLM with cited evidence. Weighted sum = Focus score.

## Tier 2: Capability Tags (25%)

MTEC structured tags present on profile:
- Diagnostics, Biologics, Drugs, Medical Devices, Digital Health Technologies, Routes of Administration
- Expertise: sepsis, hemorrhage, autonomous care, robotics, psychological health, musculoskeletal, immunology, biomanufacturing, regulatory

Scoring: fraction of high-value tags present × 100, with hemorrhage/diagnostics/sepsis weighted 2×.

## Tier 3: Dual-Use Signal (20%)

- Civilian revenue or commercial product: +30
- FDA 510(k)/PMA/De Novo: +25
- Active ClinicalTrials.gov entry: +15
- Peer-reviewed publications: +10
- Consumer/civilian version of product: +20

Cap at 100.

## Tier 4: Readiness (15%)

- TRL 4–6 (prototype to clinical): +40
- TRL 7–9 (FDA path clear): +30
- Military-experienced team: +15
- Currently seeking investment: +15

## Output schema

```json
{
  "company_id": "...",
  "composite_score": 0-100,
  "focus_area_scores": {"ccc": 0-100, "mom": 0-100, "msis": 0-100, "mid": 0-100, "crm": 0-100, "cbtr": 0-100},
  "capability_score": 0-100,
  "dual_use_score": 0-100,
  "readiness_score": 0-100,
  "evidence": ["string"],
  "flags": ["string"],
  "confidence": "high|medium|low"
}
```