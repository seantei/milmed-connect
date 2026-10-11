# Dual-Use Scoring Rubric v2

**Shared schema for rules, Jev, and LLM.** Missing evidence = "unknown," never a low score or automatic contradiction.

Composite score is calculated only from components that have a scored (non-unknown) value. Weights of available components are renormalized to 100%. If insufficient evidence remains, composite is null and confidence is "insufficient."

Base weights (renormalize when some are unknown):
- Focus Area Fit: 40%
- Capability Tags: 25%
- Dual-Use Signal: 20%
- Readiness: 15%

## Tier 1: Focus Area Fit (40%)

| Focus Area | Weight within tier | What counts |
|------------|--------------------|-------------|
| Combat Casualty Care | 28% | Hemorrhage control, tourniquets, damage-control resuscitation, en-route critical care, prolonged field care, evacuation robotics, point-of-injury diagnostics, blast/burn/TBI |
| Military Operational Medicine | 23% | Environmental physiology (heat, cold, altitude), human performance, sleep, musculoskeletal injury prevention, psychological health, suicide prevention |
| Medical Simulation & Info Sciences | 21% | Telemedicine, remote monitoring, AI decision support, medical training (AR/VR/simulation), EHR interoperability, autonomous care |
| Military Infectious Diseases | 15% | Point-of-care molecular diagnostics, antimicrobials/AMR, vaccines, wound infection, sepsis countermeasures, vector-borne |
| Clinical & Rehabilitative Medicine | 12% | Limb loss/prosthetics, tissue regeneration, vision restoration, reconstructive surgery, rehab robotics |
| Chemical & Biological Threat Reduction | 1% | Detection, prophylaxis, treatment for CBRN exposures |

Each focus area is scored 0–100 only with cited evidence; otherwise state = "unknown."

## Tier 2: Capability Tags (25%)

MTEC structured tags and expertise areas. Scored as fraction of high-value tags present, with hemorrhage/diagnostics/sepsis weighted 2×. Unknown if no tags or expertise data exist.

## Tier 3: Dual-Use Signal (20%)

Points only for affirmative evidence:
- Civilian revenue or commercial product: +30
- FDA 510(k)/PMA/De Novo: +25
- Active ClinicalTrials.gov entry: +15
- Peer-reviewed publications: +10
- Consumer/civilian version of product: +20

Cap at 100. Unknown if none of these evidence types are present.

## Tier 4: Readiness (15%)

- TRL 4–6: +40
- TRL 7–9: +30
- Military-experienced team: +15
- Currently seeking investment: +15

Unknown if no TRL, team, or seeking-investment evidence exists.

## Shared output schema

```json
{
  "company_id": "...",
  "composite_score": null,
  "focus_area_scores": {
    "combat_casualty_care": {"score": null, "state": "unknown|scored", "evidence_ids": []},
    "military_operational_medicine": {"score": null, "state": "unknown|scored", "evidence_ids": []},
    "medical_simulation_info_sciences": {"score": null, "state": "unknown|scored", "evidence_ids": []},
    "military_infectious_diseases": {"score": null, "state": "unknown|scored", "evidence_ids": []},
    "clinical_rehabilitative_medicine": {"score": null, "state": "unknown|scored", "evidence_ids": []},
    "chemical_biological_threat_reduction": {"score": null, "state": "unknown|scored", "evidence_ids": []}
  },
  "capability_score": {"score": null, "state": "unknown|scored", "evidence_ids": []},
  "dual_use_score": {"score": null, "state": "unknown|scored", "evidence_ids": []},
  "readiness_score": {"score": null, "state": "unknown|scored", "evidence_ids": []},
  "evidence": [{"id": "...", "text": "...", "source": "...", "retrieved_at": "..."}],
  "flags": [{"type": "contradiction|other", "description": "...", "evidence_ids": []}],
  "confidence": "high|medium|low|insufficient",
  "summary": "..."
}
```

**Rule for overall score:** Sum (weight × score) for scored components only, then divide by the sum of those weights. If the sum of available weights is below 40%, set composite_score to null and confidence to "insufficient."

Contradiction flags require affirmative conflicting evidence. Missing evidence is never a contradiction.
