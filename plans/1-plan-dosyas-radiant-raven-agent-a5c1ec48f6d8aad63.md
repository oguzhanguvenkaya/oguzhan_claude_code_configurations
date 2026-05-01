# Phase 1.1.11 — SKU 22870.261.001 Proposal Plan

**Plan mode aktif** — Write tool ile proposal JSON yazılması gerekiyor ama plan mode kısıtlaması var. Aşağıdaki JSON içeriği `scripts/audit/proposals/22870.261.001.json` dosyasına yazılacak (kullanıcı onayı sonrası).

## Ürün Özeti

- **Ürün**: MENZERNA Sealing Wax Protection (Improved Formulation) - 1 lt
- **Group/Sub_type**: paint_protection_quick / liquid_sealant
- **Kaynak**: Resmi Menzerna TDB (PDF) + ürün sayfası MD
- **Ürün niteliği**: Carnauba-based polishing wax/sealant, RTU, paint protection

## Kaynak Önemli Bulgular

PDF (resmi datasheet — TDB_SWPIF_22080_20250809_EN):
- Application: Rotativ, Orbital, manual
- Recommended speed: 800-1000 rpm
- Durability: "A few weeks"
- Density: 1.0 kg/l
- Color: White
- Storage: 5-30 °C
- "Sealant is quick and easy to apply, delivers a brilliant gloss in no time, and reliably protects the clear coat against environmental influences"

MD (menzerna.com ürün sayfası):
- "Service life: 1 month"
- "Carnauba-based polishing wax. The sealant provides basic protection and can be easily applied either mechanically or manually"
- "Creates a protective, dirt-repellent film. The polishing wax generates deep shine and brilliance"
- Steps: Heavy Cut → Medium Cut → Finish → Protection (bu ürün step 4)

**ÖNEMLİ**: PDF'deki SKU 22080.261.001, input SKU 22870.261.001 — büyük olasılıkla aynı ürünün farklı kataloglama kodu (improved formulation). Ürün adı birebir kaynaklardan: "Sealing Wax Protection".

## Karar Tablosu (5-Karar Prensibi)

| Key | Karar | Gerekçe |
|---|---|---|
| application_method | skip | DB="Machine or Hand" ≈ PDF "Rotativ, Orbital, manual" |
| concentrate | data_gap | Kaynakta concentrate bilgisi yok (RTU) |
| consumption_per_car_ml | data_gap | Per-pad pea drops var ama per-car ml yok |
| dilution_ratio | data_gap | RTU sealant, dilution yok |
| durability_km | data_gap | km cinsinden değer yok |
| durability_months | fill=1 | MD "Service life: 1 month" (PDF "A few weeks" MD ile uyumlu, MD daha spesifik) |
| features | update | DB tek özellik; kaynak çok daha zengin |
| filler_free | skip | Rule B: Sealing Wax için anlamlı; DB=true, kaynak çelişmiyor |
| function | fill | Kaynak metni zengin, 3-4 özellik içeren cümle yazılabilir |
| ph_level | data_gap | Kaynakta pH yok |
| ph_tolerance | data_gap | Kaynakta pH tolerance aralığı yok |
| rating_beading | data_gap | Rule F: DB null → data_gap |
| rating_durability | data_gap | Rule F: DB null → data_gap |
| rating_self_cleaning | data_gap | Rule F: DB null → data_gap |
| scent | data_gap | Koku bilgisi yok |
| target_surface | fill="paint" | "Clear coat", "car paint" — net paint ürünü |
| target_surfaces | category_mismatch | Rule A: deprecated plural |
| volume_ml | skip | DB="1000", PDF "1 liter" eşleşiyor |

**Toplam**: 1 fill listesi 2 kalem (durability_months, function, target_surface = 3 fill), 1 update, 3 skip, 9 data_gap, 1 category_mismatch.

Düzeltme: fill = 3 (durability_months, function, target_surface), update = 1 (features), skip = 3 (application_method, filler_free, volume_ml), data_gap = 10 (concentrate, consumption_per_car_ml, dilution_ratio, durability_km, ph_level, ph_tolerance, rating_beading, rating_durability, rating_self_cleaning, scent), category_mismatch = 1 (target_surfaces). Toplam = 18 ✓ (canonical_keys 18 key)

## Hazırlanan Output JSON İçeriği

```json
{
  "sku": "22870.261.001",
  "sources": ["raw-pages/22870.261.001.md", "raw-pages/22870.261.001.pdf"],
  "proposed_changes": {
    "fill": [
      {
        "key": "durability_months",
        "value": "1",
        "evidence": "MD: 'Service life: 1 month'. PDF: 'Durability: A few weeks' (uyumlu, MD daha spesifik).",
        "confidence": "medium"
      },
      {
        "key": "function",
        "value": "Carnauba bazlı boya koruma cilası; clear coat'u çevresel etkilere karşı korur, kir tutmaz koruyucu film oluşturur ve derin parlaklık ile brilliance sağlar.",
        "evidence": "MD: 'creates a protective, dirt-repellent film. The polishing wax generates deep shine and brilliance on the surface of the coat'. PDF: 'reliably protects the clear coat against environmental influences'. MD: 'carnauba-based polishing wax'.",
        "confidence": "high"
      },
      {
        "key": "target_surface",
        "value": "paint",
        "evidence": "PDF ve MD'de 'clear coat', 'car paint', 'surface of the coat' — net olarak boya/clear coat ürünü.",
        "confidence": "high"
      }
    ],
    "update": [
      {
        "key": "features",
        "current": "|Doğal parlaklık ve koruma|",
        "proposed": "|carnauba_based|deep_shine|dirt_repellent|clear_coat_protection|easy_application|brilliant_gloss|",
        "evidence": "MD: 'carnauba-based polishing wax', 'deep shine and brilliance', 'dirt-repellent film'. PDF: 'quick and easy to apply, delivers a brilliant gloss', 'reliably protects the clear coat'.",
        "confidence": "high"
      }
    ],
    "skip": [
      {
        "key": "application_method",
        "current": "Machine or Hand",
        "evidence": "PDF: 'Application: Rotativ, Orbital, manual'. MD: 'Orbital' + 'By hand'. DB değeri kaynakla uyumlu."
      },
      {
        "key": "filler_free",
        "current": "true",
        "evidence": "Rule B: Sealing Wax için filler_free anlamlı. Kaynakta filler içerdiğine dair bir ifade yok; DB=true korunabilir."
      },
      {
        "key": "volume_ml",
        "current": "1000",
        "evidence": "PDF Volumes: '1 liter 22080.261.001'. MD: '1l 22080.261.001'. 1000 ml doğru."
      }
    ],
    "data_gap": [
      {"key": "concentrate", "reason": "Kaynakta concentrate/dilute bilgisi yok; RTU sealant."},
      {"key": "consumption_per_car_ml", "reason": "PDF pad başına 3-7 pea-sized drop bilgisi var ama araç başı ml yok."},
      {"key": "dilution_ratio", "reason": "RTU ürün, kaynakta dilution oranı yok."},
      {"key": "durability_km", "reason": "Dayanıklılık km cinsinden belirtilmemiş."},
      {"key": "ph_level", "reason": "Kaynakta ürün pH değeri yok."},
      {"key": "ph_tolerance", "reason": "Kaynakta pH tolerance aralığı yok."},
      {"key": "rating_beading", "reason": "Rule F: DB null, ayrı faza bırakıldı."},
      {"key": "rating_durability", "reason": "Rule F: DB null, ayrı faza bırakıldı."},
      {"key": "rating_self_cleaning", "reason": "Rule F: DB null, ayrı faza bırakıldı."},
      {"key": "scent", "reason": "Koku bilgisi kaynakta yok."}
    ],
    "category_mismatch": [
      {"key": "target_surfaces", "reason": "Rule A: deprecated plural; target_surface kullanılıyor."}
    ]
  },
  "notes": "PDF resmi Menzerna TDB 'Sealing Wax Protection - Improved Formulation' (TDB_SWPIF_22080_20250809_EN). PDF'deki SKU 22080.261.001, input SKU 22870.261.001 ile farklı; aynı ürünün improved formulation versiyonu için iç kataloglama farkı olabilir. Ürün carnauba bazlı sealant/wax (paint_protection_quick > liquid_sealant). Hardness Rule C kapsam dışı (paint_protection_quick → category_mismatch ama canonical_keys'te hardness yok, listelenmedi). filler_free Rule B kapsamında 22870.261.001 için anlamlı tutuldu."
}
```

## Özet

```
DONE 22870.261.001: 3 fill, 1 update, 3 skip, 10 data_gap, 1 category_mismatch
```

## Onay sonrası yapılacak

1. Yukarıdaki JSON içeriğini Write tool ile şu yola yaz: `/Users/projectx/Desktop/Claude Code Projects/Products Jsons/retrieval-service/scripts/audit/proposals/22870.261.001.json`
2. Özet satırını döndür.
