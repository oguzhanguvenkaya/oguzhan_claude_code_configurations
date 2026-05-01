# Plan: SKU 22070.261.001 (MENZERNA Power Lock) — Brand Data Enrichment Proposal

## Sources Read
- inputs/22070.261.001.json
- raw-pages/22070.261.001.md
- raw-pages/22070.261.001.pdf (Technical Datasheet — PLUP)

## Key Findings From Sources
- Product: Menzerna Power Lock Ultimate Protection (PLUP) — polymer coating sealant for clear coats
- Service life: up to 6 months (matches DB `durability_months: 6`)
- Application: Orbital low-speed OR by hand (Hand Applicator / Microfiber Towel) — DB says "Machine or Hand (Apply & Buff)" — matches
- Volume: 1 L = 22070.261.001 (matches DB `volume_ml: 1000`)
- Polymer technology, deep gloss, maximum beading effect, sealant for new vehicles, environmental protection
- Density 1.0 kg/l, viscosity 4-10 Pas, color green
- Filler-free: Power Lock IS in the allow-list (Rule B), and PDF describes it as "Polymer Sealing Blend" / sealant — DB has `filler_free: true`. No explicit "filler-free" claim in raw, but PLUP is a sealant in Power Lock family — keep as skip.
- No pH info, no dilution, no consumption per car, no scent, no target surface beyond "automotive clear coats"
- No kit_contents key in canonical_keys list

## 5-Decision Categorization (canonical_keys all 18)

1. application_method — DB "Machine or Hand (Apply & Buff)"; raw confirms orbital + hand → SKIP
2. concentrate — DB null; no info in raw → DATA_GAP
3. consumption_per_car_ml — DB null; raw gives per-pad drops not per car → DATA_GAP
4. dilution_ratio — DB null; ready-to-use sealant, no dilution mentioned → DATA_GAP
5. durability_km — DB null; no km info → DATA_GAP
6. durability_months — DB "6"; raw "up to 6 Month" → SKIP
7. features — DB "|Polimer teknolojisi, derin parlaklık|" (Turkish, not snake_case); raw has clear features (polymer, beading, deep_gloss, environmental_protection, long_lasting_seal) → UPDATE to pipe-separated snake_case
8. filler_free — Rule B: Power Lock allowed; DB true; raw doesn't explicitly say but it's a polymer sealant → SKIP
9. function — DB null; Rule G: 3-4 feature sentence → FILL
10. ph_level — DB null; no pH for product itself in raw → DATA_GAP
11. ph_tolerance — DB null; no tolerance range in raw (Rule E) → DATA_GAP
12. rating_beading — DB null; Rule F → DATA_GAP
13. rating_durability — DB null; Rule F → DATA_GAP
14. rating_self_cleaning — DB null; Rule F → DATA_GAP
15. scent — DB null; no scent info → DATA_GAP
16. target_surface — DB null; raw clearly says "all automotive clear coats" / "car paint" → FILL "paint"
17. target_surfaces — Rule A: deprecated → CATEGORY_MISMATCH
18. volume_ml — DB "1000"; raw "1 L" → SKIP

Counts: 1 fill (function only) + 1 fill (target_surface) = 2 fill, 1 update (features), 4 skip (application_method, durability_months, filler_free, volume_ml), 10 data_gap (concentrate, consumption_per_car_ml, dilution_ratio, durability_km, ph_level, ph_tolerance, rating_beading, rating_durability, rating_self_cleaning, scent), 1 category_mismatch (target_surfaces) = 18 total ✓

## Proposed Output JSON (to be written to scripts/audit/proposals/22070.261.001.json)

```json
{
  "sku": "22070.261.001",
  "sources": ["raw-pages/22070.261.001.md", "raw-pages/22070.261.001.pdf"],
  "proposed_changes": {
    "fill": [
      {
        "key": "function",
        "value": "Long-lasting polymer coating sealant for automotive clear coats; up to 6 months protection with maximum beading effect; deep gloss preservation; protects against environmental influences",
        "evidence": "PDF: 'Menzerna Power Lock Ultimate Protection (PLUP) is the long-lasting polymer coating sealant for all automotive clear coats. PLUP effectively seals the coating surface for long-lasting preservation. Protects against damaging environmental influences while maintaining a brilliant deep gloss and it persuades with a maximum beading effect.' / md: 'Premium long-lasting sealant'",
        "confidence": "high"
      },
      {
        "key": "target_surface",
        "value": "paint",
        "evidence": "PDF/md: 'polymer coating sealant for all automotive clear coats' / 'Polymer sealant for all automotive clear coats'",
        "confidence": "high"
      }
    ],
    "update": [
      {
        "key": "features",
        "current": "|Polimer teknolojisi, derin parlaklık|",
        "proposed": "|polymer_technology|deep_gloss|water_beading|long_lasting_seal|environmental_protection|",
        "evidence": "PDF: 'long-lasting polymer coating sealant', 'maintaining a brilliant deep gloss', 'maximum beading effect', 'Protects against damaging environmental influences'; md: 'unbeatable beading effect', 'peerless deep shine', 'Also suitable as a long-lasting coat sealant for new vehicles'",
        "confidence": "high"
      }
    ],
    "skip": [
      {
        "key": "application_method",
        "current": "Machine or Hand (Apply & Buff)",
        "evidence": "PDF: 'Orbital: lower speed range (also manual usage with Hand Applicator or Microfiber Towel possible)'; md: 'Orbital: Low rotational speed range / By hand: Light pressure with circular motions'"
      },
      {
        "key": "durability_months",
        "current": "6",
        "evidence": "PDF: 'Service life: up to 6 Month'; md: 'Service life — 6 months'"
      },
      {
        "key": "filler_free",
        "current": "true",
        "evidence": "Power Lock is a polymer sealant (Polymer Sealing Blend per PDF subtitle); not a filler/glaze. DB value true is consistent with sealant chemistry though raw does not use the literal phrase 'filler-free'."
      },
      {
        "key": "volume_ml",
        "current": "1000",
        "evidence": "PDF Volumes: '1 L: 22070.261.001'; md: '1l | 22070.261.001'"
      }
    ],
    "data_gap": [
      {"key": "concentrate", "reason": "Raw materials do not state whether product is concentrate or ready-to-use; PDF instructions imply ready-to-use but no explicit concentrate field."},
      {"key": "consumption_per_car_ml", "reason": "PDF gives per-pad pea-sized drop counts for a 40x40cm area, not consumption per vehicle."},
      {"key": "dilution_ratio", "reason": "No dilution mentioned; product applied directly to pad."},
      {"key": "durability_km", "reason": "Service life expressed only in months (6), no km figure."},
      {"key": "ph_level", "reason": "No pH value listed in PDF chemical information (only density 1.0 kg/l, viscosity 4-10 Pas)."},
      {"key": "ph_tolerance", "reason": "No pH tolerance range listed; PDF technical sheet contains no pH data."},
      {"key": "rating_beading", "reason": "DB null; rating fields reserved for separate phase (Rule F)."},
      {"key": "rating_durability", "reason": "DB null; rating fields reserved for separate phase (Rule F)."},
      {"key": "rating_self_cleaning", "reason": "DB null; rating fields reserved for separate phase (Rule F)."},
      {"key": "scent", "reason": "No fragrance/scent information in PDF or md sources."}
    ],
    "category_mismatch": [
      {"key": "target_surfaces", "reason": "Plural deprecated per Rule A; use target_surface (singular) instead."}
    ]
  },
  "notes": "PDF (resmi datasheet) Power Lock Ultimate Protection için 'Polymer Sealing Blend' alt başlık ile tanımlıyor; sub_type liquid_sealant ile uyumlu. Renk yeşil, yoğunluk 1.0 kg/l, viskozite 4-10 Pas — ancak bu key'ler canonical_keys listesinde değil. Application: ana kullanım orbital low-speed; el ile uygulama da destekleniyor. filler_free için raw kaynakta literal 'filler-free' ifadesi yok ancak Power Lock sealant ailesinde olduğundan ve Rule B muafiyet kapsamında olduğundan DB değeri (true) korundu."
}
```

## Validation
- Total canonical_keys: 18
- Categorized: 2 fill + 1 update + 4 skip + 10 data_gap + 1 category_mismatch = 18 ✓
- No duplicates, no missing keys

## Action To Execute (after plan approval)
- Write the above JSON to: `/Users/projectx/Desktop/Claude Code Projects/Products Jsons/retrieval-service/scripts/audit/proposals/22070.261.001.json`
- Return single-line summary: `DONE 22070.261.001: 2 fill, 1 update, 4 skip, 10 data_gap, 1 category_mismatch`
