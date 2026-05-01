# Phase 1.1.11 — Brand Page + PDF Driven Data Enrichment

## Context

`paint_protection_quick` (22 ürün × 18 meta key) ve `ceramic_coating` (17 ürün × 14 meta key) kategorilerinin meta key matrisinde çoğu hücre boş — meta key'ler tanımlı ama çoğu üründe değer yok. Bu plan **bu iki kategorinin ürün verisini ana marka sayfaları + PDF teknik datasheet'ler ile zenginleştirir**.

**Hedef:** Her ürünün her canonical meta key'i için raw kaynaklardan emin değerleri DB'ye yaz, eksikleri not et, anlamsız key'leri tespit et. Kullanıcı tek tek onaylar.

> Bu görev **bağımsız** ve önce yapılır. Phase 1.1.10 (contains_sio2 fix + pH fix) bu plan tamamlandıktan **sonra** ele alınır — bekliyor.

## Kaynaklar

### URL listesi (37 ürün scrape edilecek, kullanıcı onayı)

- **paint_protection_quick.md**: 19 URL (4 INNOVACAR + 7 GYEON + 3 MENZERNA + 4 FRA-BER + 1 GYEON Q Wax)
- **ceramic_coatings.md**: 18 URL (14 GYEON + 3 MX-PRO + 1 INNOVACAR; 1 URL `interior_cleaner` kategorisinde — Q2M-PM500M, raw kaydedilir, analiz edilmez)

### PDF teknik datasheet (3/3, MENZERNA)

| PDF | SKU | Ürün |
|---|---|---|
| `Products Jsons/sealingwax.pdf` | 22870.261.001 | MENZERNA Sealing Wax |
| `Products Jsons/powerlock.pdf` | 22070.261.001 | MENZERNA Power Lock |
| `Products Jsons/ceramic_spray_sealant.pdf` | 26919.271.001 | MENZERNA Ceramic Spray Sealant |

PDF'ler resmi Menzerna Technical Datasheet formatında — yapılandırılmış (Service Life, Storage, Density, Polishing tools, Recommended Usage steps, Polishing Programm matrix). Read tool ile native okunuyor.

**PDF öncelik kuralı**: 3 MENZERNA ürününde URL ↔ PDF çelişki olursa PDF (resmi datasheet) öncelikli.

## Defuddle CLI test sonuçları (4 domain ✓ uyumlu)

| Domain | Test | Çıktı kalitesi |
|---|---|---|
| `gyeon.co` | cancoat-evo | ✓ Mükemmel — technical details + application steps |
| `menzerna.com` | ceramic-spray-sealant | ✓ İyi — processing + service life + steps |
| `shop.fra-ber.it` | lustrawax | ✓ İyi (image link post-process gerek) |
| `mtskimya.com` | mx-pro-diamond-plus | ✓ İyi (Türkçe — dayanım + hardness + kit_contents) |
| `en.innovacar.it` | henüz test edilmedi (Faz A başında doğrulanır) | beklenen ✓ |

## URL → SKU eşleşmesi (final, kullanıcı onaylı)

> Boyut/variant uyuşmazlığı kontrolü yok (kullanıcı: "ürün adı aynı yeterli, boyut sonra").

### paint_protection_quick — 19 ürün scrape + analiz

| # | URL | SKU | Brand+Name | PDF |
|---|---|---|---|---|
| 1 | en.innovacar.it/.../w1-quick-detailer | 79301 | INNOVACAR W1 QUICK DETAILER | – |
| 2 | en.innovacar.it/.../sc0-hydro-sealant | 700097 | INNOVACAR SC0 HYDRO SEALANT | – |
| 3 | en.innovacar.it/.../sc1-sealant | 700096 | INNOVACAR SC1 SEALANT | – |
| 4 | en.innovacar.it/.../h2o-coat | 79304 | INNOVACAR H2O COAT | – |
| 5 | gyeon.co/.../ppf-maintain-redefined | Q2M-PPFMR500M | GYEON QM PPF Maintain | – |
| 6 | gyeon.co/.../ceramicdetailer | Q2M-CDYA1000M | GYEON QM CeramicDetailer | – |
| 7 | gyeon.co/.../cure-redefined | Q2M-CRYA250M | GYEON QM Cure REDEFINED | – |
| 8 | gyeon.co/.../cure-matte-redefined | Q2M-CMR500M | GYEON QM Cure Matte | – |
| 9 | gyeon.co/.../quickview | Q2-QV120M | GYEON Q QuickView | – |
| 10 | gyeon.co/.../quickdetailer | Q2M-QDYA1000M | GYEON QM QuickDetailer | – |
| 11 | gyeon.co/.../wetcoat | Q2M-WCYA4000M | GYEON QM WetCoat | – |
| 12 | menzerna.com/.../sealing-wax-protection | 22870.261.001 | MENZERNA Sealing Wax | sealingwax.pdf ✓ |
| 13 | menzerna.com/.../power-lock-ultimate-protection | 22070.261.001 | MENZERNA Power Lock | powerlock.pdf ✓ |
| 14 | menzerna.com/.../ceramic-spray-sealant | 26919.271.001 | MENZERNA Ceramic Spray Sealant | ceramic_spray_sealant.pdf ✓ |
| 15 | shop.fra-ber.it/.../lustrawax | 74059 | FRA-BER Lustrawax | – |
| 16 | shop.fra-ber.it/.../nanotech | 74062 | FRA-BER Nanotech | – |
| 17 | shop.fra-ber.it/.../lustratouch | 75182 | FRA-BER Lustratouch | – |
| 18 | shop.fra-ber.it/.../lustradry | 75016 | FRA-BER Lustradry | – |
| 19 | gyeon.co/.../wax | Q2-W175G | GYEON Q Wax | – |

**Atlanan 3 SKU (kullanıcı kararı, kapsam dışı):** 70545, 71331, 78779 (FRA-BER konsantre)

### ceramic_coating — 17 ürün scrape + analiz

| # | URL | SKU | Brand+Name |
|---|---|---|---|
| 1 | gyeon.co/.../q2-purify-coat | Q2-PC100M | GYEON Q PurifyCoat |
| 2 | gyeon.co/.../one-evo | Q2-OLE100M | GYEON Q One EVO |
| 3 | gyeon.co/.../mohs-evo | Q2-MLE100M | GYEON Q Mohs EVO |
| 4 | gyeon.co/.../pure-evo | Q2-PLE50M | GYEON Q Pure EVO |
| 5 | gyeon.co/.../syncro-evo | Q2-SLE50M | GYEON Q Syncro EVO |
| 6 | gyeon.co/.../cancoat-evo | Q2-CCE200M | GYEON Q CanCoat EVO |
| 7 | gyeon.co/.../matte-evo | Q2-MTEL50M | GYEON Q Matte EVO |
| 8 | gyeon.co/.../ppf-evo | Q2-PPFE50M | GYEON Q PPF EVO |
| 9 | gyeon.co/.../trim-evo | Q2-TRE30M | GYEON Q Trim EVO |
| 10 | gyeon.co/.../rim-evo | Q2-RE30M | GYEON Q Rim EVO |
| 11 | gyeon.co/.../leathershield-evo | Q2-LSE50M | GYEON Q Leather Shield EVO |
| 12 | gyeon.co/.../antifog | Q2-AF120M | GYEON Q AntiFog |
| 13 | gyeon.co/.../view-evo | Q2-VE20M | GYEON Q View EVO |
| 14 | mtskimya.com/.../mx-pro-diamond-plus | MXP-DPCN50KS | MX-PRO DIAMOND Plus |
| 15 | mtskimya.com/.../mx-pro-crystal | MXP-CCN50KS | MX-PRO CRYSTAL |
| 16 | mtskimya.com/.../mx-pro-hydro | MXP-HC50KS | MX-PRO HYDRO |
| 17 | en.innovacar.it/.../sc3-glass-sealant | 79296 | INNOVACAR SC3 GLASS SEALANT |

### interior_cleaner — 1 ürün (sadece raw scrape, analiz Phase 1.1.12)

| URL | SKU |
|---|---|
| gyeon.co/.../q2m-purify-maintain | Q2M-PM500M (template_group=interior_cleaner) |

**Toplam Faz A scrape: 37 URL** | **Toplam Faz B analiz: 36 ürün**

## 3-Faz Plan

### Faz A — Scraping + PDF copy pipeline (~1 saat)

**Script:** `retrieval-service/scripts/phase-1-1-11-collect-sources.ts`

```ts
// 1. URL→SKU mapping inline (yukarıdaki 37 satır)
// 2. Her URL için: defuddle parse <url> --md
// 3. Post-process: image link regex temizleme, çoklu boş satır collapse
// 4. Kaydet: scripts/audit/raw-pages/<sku>.md
// 5. PDF copy: 3 MENZERNA PDF'i scripts/audit/raw-pages/<sku>.pdf'e
// 6. Audit özet: scripts/audit/_collect-summary.json
```

```bash
bun run scripts/phase-1-1-11-collect-sources.ts
```

**Çıktı:**
- `raw-pages/<sku>.md` — 37 markdown
- `raw-pages/<sku>.pdf` — 3 PDF kopyası
- `_collect-summary.json` — audit özeti

### Faz B — Subagent-driven per-product analysis (~3-5 saat, paralel batch)

**36 ürün** (Q2M-PM500M hariç) için her ürüne bir general-purpose Agent.

#### Prensip — Her canonical key için sırayla 5 karar

Subagent **kategorinin TÜM canonical key'lerini** sırayla tarar (sadece bir key'e değil — tüm 14-18 key her ürün için kontrol). Her key için 5 olası karar:

| Karar | Ne zaman | Action |
|---|---|---|
| **fill** | DB değer yok (null/missing) + kaynaklarda emin değer var | yeni değer öner + evidence + confidence |
| **update** | DB değer var ama kaynaklarda farklı/güncel değer var | mevcut → öneri + evidence |
| **skip** | DB değer var ve kaynaklarla eşleşiyor | no-op |
| **data_gap** | DB değer yok ve kaynaklarda da yok / agent emin değil | "doldurulamadı" not |
| **category_mismatch** | Bu key o ürün için **anlamsız** (örn paste_wax'te dilution_ratio, antibacterial_coating'te beading rating) | "bu key bu üründe olmamalı" not, kullanıcı karar versin |

**Confidence eşiği:** Sadece **emin** değerler `fill/update`'e girer. Kaynak metinde birebir/açık çıkarım varsa yaz; "muhtemelen" düzeyindeyse `data_gap`'e at.

**Validasyon:** Her canonical key 5 listeden birine girmeli — kayıp key olmamalı.

#### Subagent input paketi (5 parça)

1. **Raw markdown**: `raw-pages/<sku>.md`
2. **(varsa) PDF**: `raw-pages/<sku>.pdf` (3 MENZERNA için, Read tool)
3. **DB current meta** — agent neyi araması gerektiğini bilsin:
   ```json
   {
     "sku": "...", "name": "...", "brand": "...",
     "template_group": "...", "template_sub_type": "...",
     "current_meta": {
       "volume_ml": 200,
       "durability_months": null,
       "ph_tolerance": null,
       "...": "..."
     }
   }
   ```
4. **Canonical key listesi** (kategorinin tüm 14-18 key'i) — "aranan kelimeler" listesi
5. **Talimat**:
   > "Bu ürünün raw markdown + (varsa) PDF kaynaklarını oku. Aşağıdaki canonical key listesini SIRAYLA tara — her key için kararın ne (fill/update/skip/data_gap/category_mismatch). Sadece **emin olduğun** değerleri fill/update et — şüphedeysen data_gap. Bir key bu ürün için anlamsız geliyorsa category_mismatch + gerekçe. Ürün adı uydurma; kaynak metinden birebir oku."

#### Subagent output JSON

```json
{
  "sku": "26919.271.001",
  "sources": ["raw-pages/26919.271.001.md", "raw-pages/26919.271.001.pdf"],
  "proposed_changes": {
    "fill": [
      {"key": "durability_months", "value": 3,
       "evidence": "PDF: Service Life: ca. 3 Month",
       "confidence": "high"},
      {"key": "application_method", "value": "Microfiber cloth",
       "evidence": "PDF: Polishing tools: Microfiber cloth",
       "confidence": "high"}
    ],
    "update": [
      {"key": "features", "current": "|x|y|", "proposed": "|x|y|water-repellent|",
       "evidence": "PDF: extremely dirt-repellent ... water-repellent effect",
       "confidence": "medium"}
    ],
    "skip": [
      {"key": "volume_ml", "current": 500,
       "evidence": "PDF: 500 ml: 26919.271.001"}
    ],
    "data_gap": [
      {"key": "ph_tolerance", "reason": "Kaynaklarda pH bilgisi yok"},
      {"key": "rating_durability", "reason": "Sayısal rating yok"}
    ],
    "category_mismatch": [
      {"key": "scent", "reason": "MENZERNA datasheet'inde koku belirtilmemiş; sealant ürününde scent normalde olmamalı"}
    ]
  },
  "notes": "PDF'de Polishing Programm matrix var — Step 4 Protection."
}
```

**Batch:** 5-7 paralel general-purpose Agent

**Çıktı:** `scripts/audit/proposals/<sku>.json` (36 dosya)

### Faz C — Onay + uygulama (kullanıcı tempoyla)

**Step C.1** — Birleştirme:
```ts
// scripts/audit/proposals/*.json → scripts/audit/_review-table.md
// Kompakt markdown: ürün başlığı + 5 blok (fill/update/skip/data_gap/category_mismatch)
// Her satıra checkbox prefix: [ ] (sen edit ile [x] yapacaksın)
```

**Step C.2** — Kullanıcı incelemesi:
- Tek tek değişiklikleri inceler
- Onayladığı önerilerin checkbox'ını işaretler
- Reddettikleri [ ] kalır → atılır

**Step C.3** — DB update script:
```ts
// scripts/phase-1-1-11-apply-enrichment.ts --apply
// Transaction + row-count guard
// jsonb_set ile specs[key] = value (COALESCE specs '{}', create_missing=true)
// category_mismatch onaylandıysa: ilgili row product_meta'dan SİL + specs'ten key DELETE
// Audit: before/after DB'den okur, _apply-audit.json
```

**Step C.4** — EAV re-project + verification:
```bash
bun run scripts/project-specs-to-meta.ts
```

SKU bazlı SQL kontrol — onaylanan key'ler product_meta'da görünüyor mu, kaldırılanlar gitti mi.

## Açık sorular (implementation öncesi)

1. **Subagent paralel batch boyutu**:
   - 5-7 paralel (önerilen) → 36 ürün için 5-7 batch, hızlı
   - 3-4 → güvenli ama yavaş
   - 10+ → rate limit/maliyet riski

2. **Onay akışı**:
   - (a) Tek mega `_review-table.md` checkbox — sen tüm 36 ürünü tek dosyada onayla (önerilen)
   - (b) Batch onay — 5'erli grup
   - (c) Webchat-style tek tek (en yavaş)

## Kritik dosyalar (oluşturulacak)

- `retrieval-service/scripts/phase-1-1-11-collect-sources.ts` — Faz A
- `retrieval-service/scripts/phase-1-1-11-apply-enrichment.ts` — Faz C
- `retrieval-service/scripts/audit/raw-pages/<sku>.md|pdf` — Faz A çıktısı
- `retrieval-service/scripts/audit/proposals/<sku>.json` — Faz B çıktısı
- `retrieval-service/scripts/audit/_review-table.md` — Faz C input (kullanıcı edit eder)
- `retrieval-service/scripts/audit/_apply-audit.json` — Faz C apply çıktısı

## Verification (kabul kriterleri)

| # | Kontrol | Beklenen |
|---|---|---|
| 1 | `ls scripts/audit/raw-pages/*.md \| wc -l` | 37 |
| 2 | Her raw-pages/<sku>.md > 1 KB | ✓ |
| 3 | 3 MENZERNA PDF mevcut | ✓ |
| 4 | Her ürün için bir `proposals/<sku>.json` (36 dosya) | ✓ |
| 5 | Onaylanan key'ler DB'de görünüyor (SKU bazlı SELECT) | ✓ |
| 6 | category_mismatch onaylananlar product_meta'dan silindi | ✓ |
| 7 | Reddedilen öneriler DB'ye yazılmadı | ✓ |
| 8 | CSV re-run sonrası null hücre sayısı belirgin azaldı | %50+ doluluk artışı |

## Out of Scope

- 3 atlanan paint_protection_quick SKU (70545, 71331, 78779) — ayrıca planlanır
- Q2M-PM500M analiz — raw scrape var, analiz Phase 1.1.12
- Diğer kategoriler (car_shampoo, abrasive_polish, vb.) — Phase 1.1.12+
- Yeni canonical key ekleme (mevcut 18+14 key set'i sabit)
- Embeddings/search-text yeniden hesaplama (otomatik)
- target_surface vs target_surfaces taxonomy normalize (ayrı küçük migration)
- Boyut/variant uyuşmazlığı düzeltme (kullanıcı: "sonra")
- **Phase 1.1.10 (contains_sio2 fix + pH fix)** — ayrı görev, bu plan tamamlandıktan sonra ele alınır

## Notlar

- **PDF öncelik kuralı**: 3 MENZERNA ürününde URL ↔ PDF çelişkide PDF (resmi datasheet) öncelikli
- **Subagent input vurgusu**: agent neyi arayacağını bilsin diye **DB current meta + canonical key listesi** mutlaka verilir
- **5-karar prensibi**: subagent her key için fill/update/skip/data_gap/category_mismatch'ten birini seçer; kayıp key olamaz
- **Confidence**: emin değerler fill/update'e, şüpheli veriler data_gap'e — kalite > kapsam
- **SKU SSOT prensibi**: subagent ürün adı uydurmaz, raw kaynaktan birebir okur
- FRA-BER + MTS Kimya İtalyan/Türk sayfaları — dil farkı subagent için sorun değil
- ~36 ürün × ortalama 8-12 key öneri = ~300-450 öneri review edilecek; tek mega checkbox dosya pratik
