# Phase 1.1.11 Brand Data Enrichment — SKU 26919.271.001 (Proposal JSON)

## Hedef Dosya
`/Users/projectx/Desktop/Claude Code Projects/Products Jsons/retrieval-service/scripts/audit/proposals/26919.271.001.json`

Proposals dizini mevcut (Q2-CCE200M.json zaten var). Aynı şemayla yeni dosya yazılacak.

## Yazılacak FULL Proposal JSON

```json
{
  "sku": "26919.271.001",
  "sources": ["raw-pages/26919.271.001.md", "raw-pages/26919.271.001.pdf"],
  "proposed_changes": {
    "fill": [
      {
        "key": "application_method",
        "value": "|spray_onto_microfiber|wipe_on_surface|",
        "evidence": "PDF Recommended Usage: '3. Spray two to three sprays onto the Menzerna Premium Microfiber Cloth (blue). 4. Spread the product with the Menzerna Premium Microfiber Cloth on the surface to be sealed. 5. Remove the remaining product after application on the surface with the dry side of the Menzerna Premium Microfiber Cloth (blue).' MD: 'Spray two to three sprays onto the microfiber cloth (blue), Spread with the microfiber cloth on the surface to be sealed, Remove residues with the dry side.'",
        "confidence": "high"
      },
      {
        "key": "function",
        "value": "Universally applicable spray sealant that protects all vehicle paint surfaces from environmental influences and deposits, provides very high water-repellent (beading/sheeting) effect, and simplifies subsequent cleaning of the vehicle.",
        "evidence": "MD: 'Easy and quick to apply spray sealant with very high water-repellent effect. The spray sealant protects the surface from environmental influences and deposits and is extremely dirt-repellent.' PDF Product Description: 'Menzerna Spray Sealant is a simple and quick to use spray sealant. It is universally applicable and suitable for all vehicles. Menzerna Spray Sealant protects the surface from environmental influences and deposits. It also simplifies the cleaning process.'",
        "confidence": "high"
      },
      {
        "key": "target_surface",
        "value": "paint",
        "evidence": "PDF Polishing Programm chart places product in 'Step 4 Protection' as a paint sealant alongside Power Lock Ultimate, Liquid Carnauba, Sealing Wax. PDF Product Description: 'universally applicable and suitable for all vehicles' (vehicle paint context). MD: 'Universal spray sealant for all vehicles' under paint protection workflow.",
        "confidence": "high"
      }
    ],
    "update": [
      {
        "key": "features",
        "current": "|Yüksek su iticilik, kolay uygulama|",
        "proposed": "|high_water_repellency|easy_application|quick_application|universal_use|environmental_protection|dirt_repellent|",
        "evidence": "DB current is Turkish free-text comma-separated single token, not the canonical pipe-separated lowercase snake_case format. Source evidence — MD: 'Easy and quick to apply spray sealant with very high water-repellent effect ... extremely dirt-repellent ... unbeatable product for every user in all areas of vehicle detailing.' PDF: 'simple and quick to use ... universally applicable and suitable for all vehicles ... protects the surface from environmental influences and deposits ... simplifies the cleaning process.'",
        "confidence": "high"
      }
    ],
    "skip": [
      {
        "key": "durability_months",
        "current": "3",
        "source_value": "3",
        "evidence": "MD 'Service life: 3 months'. PDF General Information: 'Service Life: ca. 3 Month'. DB value matches source.",
        "confidence": "high"
      },
      {
        "key": "volume_ml",
        "current": "500",
        "source_value": "500",
        "evidence": "MD Volumes table: '500ml | 26919.271.001'. PDF Volumes: '500 ml: 26919.271.001'. DB value matches source.",
        "confidence": "high"
      }
    ],
    "data_gap": [
      {
        "key": "concentrate",
        "reason": "Boolean key. Sources do not explicitly state concentrate vs ready-to-use. Although a spray sealant is typically RTU, neither MD nor PDF contains an explicit 'concentrate: no' or 'ready to use' statement. Per rule J, leave as data_gap."
      },
      {
        "key": "consumption_per_car_ml",
        "reason": "PDF says 'two to three sprays onto the microfiber cloth' but no ml-per-car or coverage figure is provided in either MD or PDF."
      },
      {
        "key": "dilution_ratio",
        "reason": "No dilution ratio mentioned in either source; product is sprayed directly onto microfiber cloth without dilution instructions, but explicit '1:0' / 'undiluted' wording is absent."
      },
      {
        "key": "durability_km",
        "reason": "No km-based durability specification in either MD or PDF (only month-based service life of ca. 3 months)."
      },
      {
        "key": "ph_level",
        "reason": "PDF Chemical Information lists only 'Density: 0,9 kg/l'; no pH value provided. MD does not mention pH either."
      },
      {
        "key": "ph_tolerance",
        "reason": "No pH tolerance range stated in either source. Per rule E, requires explicit numeric range; not present."
      },
      {
        "key": "rating_beading",
        "reason": "DB null. Per rule F, rating_* keys are reserved for a separate phase — proposal must remain data_gap and never fill/update."
      },
      {
        "key": "rating_durability",
        "reason": "DB null. Per rule F, rating_* keys are reserved for a separate phase — proposal must remain data_gap and never fill/update."
      },
      {
        "key": "rating_self_cleaning",
        "reason": "DB null. Per rule F, rating_* keys are reserved for a separate phase — proposal must remain data_gap and never fill/update."
      },
      {
        "key": "scent",
        "reason": "PDF lists Colour as 'Transparent' but no scent/odour information is provided in either MD or PDF."
      }
    ],
    "category_mismatch": [
      {
        "key": "target_surfaces",
        "reason": "Per rule A: target_surfaces (plural) is deprecated. Use target_surface (singular) instead. Marked category_mismatch for every product."
      },
      {
        "key": "filler_free",
        "reason": "Per rule B: filler_free is meaningful only for polish-correction products like Power Lock (22070.261.001) and Sealing Wax (22870.261.001). This SKU is Ceramic Spray Sealant (paint_protection_quick / spray_sealant) — filler_free does not apply."
      }
    ]
  },
  "notes": "MENZERNA Ceramic Spray Sealant 500ml (paint_protection_quick / spray_sealant). PDF technical datasheet is primary source (rule K). 18 canonical_keys fully partitioned: 3 fill + 1 update + 2 skip + 10 data_gap + 2 category_mismatch = 18. Rule C (hardness) and Rule D (kit_contents) not applicable — those keys are absent from canonical_keys for this SKU. Features update converts non-canonical Turkish comma-separated string into canonical pipe-separated lowercase snake_case format with 6 attributes evidenced by both sources."
}
```

## 5-Karar Validasyonu
- canonical_keys (18): application_method, concentrate, consumption_per_car_ml, dilution_ratio, durability_km, durability_months, features, filler_free, function, ph_level, ph_tolerance, rating_beading, rating_durability, rating_self_cleaning, scent, target_surface, target_surfaces, volume_ml
- Toplam: 3 fill + 1 update + 2 skip + 10 data_gap + 2 category_mismatch = 18 ✓

## Özel Kural Kontrolü
- A) target_surfaces → category_mismatch ✓
- B) filler_free Ceramic Spray Sealant → category_mismatch ✓
- C) hardness: canonical_keys'te YOK → uygulanmıyor
- D) kit_contents: canonical_keys'te YOK → uygulanmıyor
- E) ph_tolerance: kaynakta yok → data_gap ✓
- F) rating_* (3): DB null → data_gap (fill/update YOK) ✓
- G) function: 3-4 özellikli cümle ✓
- H) durability_months: "3 month" → DB "3" eşleşiyor → skip ✓
- I) features: pipe-separated lowercase snake_case ✓
- J) concentrate boolean: net belirtilmemiş → data_gap ✓
- K) PDF öncelikli (PDF'te kullanım adımları, density, polishing tools detayı var; MD ile çelişki yok) ✓

## Çıktı Sonrası Özet Satırı
`DONE 26919.271.001: 3 fill, 1 update, 2 skip, 10 data_gap, 2 category_mismatch`

## Yapılacak Tek İş
Yukarıdaki JSON'u Write tool ile şu yola yazmak:
`/Users/projectx/Desktop/Claude Code Projects/Products Jsons/retrieval-service/scripts/audit/proposals/26919.271.001.json`
