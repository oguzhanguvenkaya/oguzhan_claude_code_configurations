# Plan: SKU 700097 (INNOVACAR SC0 HYDRO SEALANT) — Phase 1.1.11 Brand Data Enrichment

## Bağlam
- Ürün: INNOVACAR SC0 HYDRO SEALANT 4.54 L (spray sealant)
- template_group: paint_protection_quick
- template_sub_type: spray_sealant
- Plan mode aktif — Write işlemi YAPMADIM. Aşağıda analiz + planlanan output JSON içeriği.

## Key-by-key analiz (5-karar prensibi)

### 1. application_method — DB null
- Raw: "Apply 2-3 sprays on the low pile microfibre cloth ... apply panel by panel" + "use the product undiluted"
- Karar: **fill** → "spray_on_microfiber_then_wipe"

### 2. concentrate — DB null
- Raw: "Use: use the product undiluted" → ready-to-use, konsantre değil
- Karar: **fill** → false

### 3. consumption_per_car_ml — DB null
- Raw: "Average consumption: 30 ml per vehicle"
- Karar: **fill** → 30

### 4. dilution_ratio — DB null
- Raw: "use the product undiluted"
- Karar: **fill** → "ready_to_use"

### 5. durability_km — DB null
- Raw'da km değeri yok
- Karar: **data_gap**

### 6. durability_months — DB="9"
- Raw: "lasts up to 5 months alone, up to 9 months with W1 + H20 maintenance"
- DB değer (9) raw'daki maksimum ile uyumlu
- Karar: **skip**

### 7. features — DB="|Su bazlı, klasik araçlar için güvenli|"
- Raw: water-based, nanotechnology, mirror_effect, color_depth, water_repellent, silk_effect, classic_car_safe, siloxanes, fluoropolymers, no_petroleum_distillates
- DB eksik özellikler içeriyor — güncellenebilir
- Karar: **update** → "|water_based|nanotechnology|mirror_effect|color_depth|water_repellent|silk_effect|classic_car_safe|high_temperature_resistant|"

### 8. filler_free — ÖZEL KURAL B
- 700097 paint_protection_quick → **category_mismatch**

### 9. function — DB null
- Raw'dan: "Water-based nanotechnological spray sealant that creates a chemical and molecular barrier on car paintwork using siloxanes and fluoropolymers, providing mirror effect, color depth, water repellency and silk-touch surface lasting up to 9 months."
- Karar: **fill** (3-4 özellikli cümle)

### 10. ph_level — DB null
- Raw: "**pH**: 7.00"
- Karar: **fill** → "7.00"

### 11. ph_tolerance — ÖZEL KURAL E
- Raw: "protecting bodywork from alkaline and acidic cleaning products" — sadece resistant ifadesi var, aralık YOK
- ph_level (7.00) ph_tolerance ile karıştırılmamalı
- Karar: **data_gap**

### 12. rating_beading — ÖZEL KURAL F
- DB null, INNOVACAR Summary'de "Beading: 5/5" var ama kural: SKIP veya data_gap
- Karar: **data_gap** (notes'ta belirt)

### 13. rating_durability — ÖZEL KURAL F
- DB null, raw'da "Duration: 5/5" var ama kural: SKIP veya data_gap
- Karar: **data_gap** (notes'ta belirt)

### 14. rating_self_cleaning — ÖZEL KURAL F
- DB null, raw'da "Self-cleaning capacity: 5/5" var ama kural: SKIP veya data_gap
- Karar: **data_gap** (notes'ta belirt)

### 15. scent — DB null
- Raw'da koku bilgisi yok
- Karar: **data_gap**

### 16. target_surface (singular) — DB null
- Raw: "suitable for body and paintwork", "suitable for all surfaces", "directly on painted surfaces"
- Karar: **fill** → "paint"

### 17. target_surfaces (plural) — ÖZEL KURAL A
- Karar: **category_mismatch**

### 18. volume_ml — DB="4540"
- Raw: "4.54 L" = 4540 ml ✓
- Karar: **skip**

## Tally
- fill: 7 (application_method, concentrate, consumption_per_car_ml, dilution_ratio, function, ph_level, target_surface)
- update: 1 (features)
- skip: 2 (durability_months, volume_ml)
- data_gap: 6 (durability_km, ph_tolerance, rating_beading, rating_durability, rating_self_cleaning, scent)
- category_mismatch: 2 (filler_free, target_surfaces)
- TOPLAM: 18 ✓ (canonical_keys uzunluğu 18)

## Planlanan Output JSON (Write yapılacak path: scripts/audit/proposals/700097.json)

```json
{
  "sku": "700097",
  "sources": ["raw-pages/700097.md"],
  "proposed_changes": {
    "fill": [
      {"key": "application_method", "value": "spray_on_microfiber_then_wipe", "evidence": "Apply 2-3 sprays of SC0 Hydro Sealant on the low pile microfibre cloth ... Apply panel by panel"},
      {"key": "concentrate", "value": false, "evidence": "Use: use the product undiluted"},
      {"key": "consumption_per_car_ml", "value": 30, "evidence": "Average consumption: 30 ml per vehicle"},
      {"key": "dilution_ratio", "value": "ready_to_use", "evidence": "use the product undiluted"},
      {"key": "function", "value": "Water-based nanotechnological spray sealant that creates a chemical and molecular barrier on car paintwork using siloxanes and fluoropolymers, providing mirror effect, color depth and water-repellent silk-touch protection lasting up to 9 months.", "evidence": "Innovacar's SC0 Hydro Sealant is a water-based nanotechnology car spray sealant ... siloxanes and fluoropolymers ... mirror effect and colour depth ... water repellent ... silk effect ... up to 9 months"},
      {"key": "ph_level", "value": "7.00", "evidence": "Summary technical data: pH: 7.00"},
      {"key": "target_surface", "value": "paint", "evidence": "suitable for body and paintwork ... can be used directly on painted surfaces"}
    ],
    "update": [
      {"key": "features", "old": "|Su bazlı, klasik araçlar için güvenli|", "new": "|water_based|nanotechnology|mirror_effect|color_depth|water_repellent|silk_effect|classic_car_safe|high_temperature_resistant|", "evidence": "water-based nanotechnology ... mirror effect and colour depth ... water repellent ... silk effect ... suitable for cars with and without coating, perfect for classic cars ... absence of petroleum distillates, suitable to withstand high temperatures"}
    ],
    "skip": [
      {"key": "durability_months", "current": "9", "evidence": "protection can last up to 9 months, if combined with W1 Quick Detailer ... and H20 Coat"},
      {"key": "volume_ml", "current": "4540", "evidence": "700097 - SC0 HYDRO SEALANT 4.54 L"}
    ],
    "data_gap": [
      {"key": "durability_km", "reason": "raw'da km değeri yok"},
      {"key": "ph_tolerance", "reason": "Sadece 'protects from alkaline and acidic cleaning products' ifadesi var; aralık (X-Y) yok. ph_level (7.00) ürünün kendi pH'ı, ph_tolerance ile karıştırılmadı."},
      {"key": "rating_beading", "reason": "INNOVACAR Summary technical data bloğunda 'Beading: 5/5' mevcut, ayrı faz"},
      {"key": "rating_durability", "reason": "INNOVACAR Summary technical data bloğunda 'Duration: 5/5' mevcut, ayrı faz"},
      {"key": "rating_self_cleaning", "reason": "INNOVACAR Summary technical data bloğunda 'Self-cleaning capacity: 5/5' mevcut, ayrı faz"},
      {"key": "scent", "reason": "raw'da koku/parfüm bilgisi yok"}
    ],
    "category_mismatch": [
      {"key": "filler_free", "reason": "Özel kural B: 700097 paint_protection_quick → anlamsız"},
      {"key": "target_surfaces", "reason": "Özel kural A: plural form her ürün için category_mismatch"}
    ]
  },
  "notes": "INNOVACAR Summary technical data bloğu mevcut (Self-cleaning 5/5, Duration 5/5, Beading 5/5, Colour depth 5/5) — rating_* alanları için ayrı faz kuralı (F) gereği data_gap olarak işaretlendi. ph_level (7.00) ürünün kendi pH'ı; ph_tolerance ile karıştırılmadı (kural E). filler_free paint_protection_quick için anlamsız (kural B). target_surfaces plural form kategorik olarak yok sayıldı (kural A). features lowercase snake_case pipe-separated formata çevrildi (kural I). Ürün adı raw kaynaktan birebir okundu, uydurma yapılmadı."
}
```

## Onay sonrası yapılacak (plan mode kapanınca)
1. Yukarıdaki JSON'u `/Users/projectx/Desktop/Claude Code Projects/Products Jsons/retrieval-service/scripts/audit/proposals/700097.json` yoluna Write
2. Özet rapor: "DONE 700097: 7 fill, 1 update, 2 skip, 6 data_gap, 2 category_mismatch"
