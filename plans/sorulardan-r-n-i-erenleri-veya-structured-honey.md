# Gyeon FAQ → productFaqTable İngestion Planı

## Context

`docs/gyeon-faqs.json` içinde Zendesk'ten kazınmış **752 İngilizce Gyeon FAQ** var (5 kategori, 100 section). Bunları `productFaqTable`'a, ya gerçek ürün SKU'suna ya da `_BRAND:gyeon:<alt>` sentinel'ine bağlı şekilde, **Türkçe çeviri ile** eklemek istiyoruz.

**Neden bu konvansiyon zaten doğru:**
- `productFaqTable` schema sadece `{sku, question, answer}` (`src/tables/product-faq.ts:10-20`).
- 3 SKU konvansiyonu zaten tanımlı (`src/conversations/index.ts:274-275`):
  1. `<real_sku>` → ürüne özel
  2. `_CAT:<group>` → kategori rehberi
  3. `_BRAND:menzerna:<cat>` → marka rehberi
- M.2-M.8 commit'leri (Apr 19) ile **varyant konsolidasyonu yapılmış**: 622 → 511 primary row. Her ürün hattının tek bir primary SKU'su var, varyantlar `variant_skus` (pipe-ayrık) + `sizes` (JSON) içinde tutuluyor. **FAQ → SKU eşlemesi artık 1:1.**
- Menzerna emsali var (74 brand FAQ, Türkçe çeviri ile, `commit b0daad0`, `scripts/seed-menzerna-brand-faqs.ts`).

**Kullanıcı tercihleri:**
- Dil: LLM ile Türkçe'ye çevir (ürün adlarını verbatim koru: "Q² Mohs EVO", "Q²M Bathe+").
- Non-product sentinel: `_CAT:*` / `_BRAND:gyeon:*` (mevcut konvansiyonu genişlet).
- Section→SKU eşlemesi: Subagent-driven, 5-6 paralel agent ile %100 doğru, eşleşmeyenleri atla.

---

## Çıktı dosyaları

| Yol | İçerik |
|---|---|
| `docs/gyeon-section-sku-map.json` | 100 section → primary SKU + barcode + confidence (subagent çıktısı, manuel review) |
| `output/csv/gyeon_faqs.csv` | Seed CSV: `sku,question,answer` (Türkçe) |
| `docs/gyeon-faqs-en-tr.jsonl` | Audit kaydı: orijinal EN + Türkçe çeviri (rollback / spot-check için) |
| `scripts/build-gyeon-sku-map.ts` | Aday eşlemeyi üreten script (Phase 1) |
| `scripts/translate-gyeon-faqs.ts` | LLM çeviri scripti (Phase 3) |
| `scripts/seed-gyeon-faqs.ts` | Seed scripti (Phase 4), `seed-menzerna-brand-faqs.ts` mirror'ı |

---

## Adımlar

### Phase 1 — Section → primary SKU eşlemesi (subagent-driven, paralel)

1. **Gyeon ürün kataloğunu dump et** (read-only).
   - `Product Groups/*.json` üzerinden brand=GYEON veya SKU prefix `Q2-`/`Q2M-`/`Q2R-` olanları topla.
   - Her ürün için: `{primary_sku, product_name, base_name, barcode, variant_skus, template_sub_type, short_description (ilk 200 char)}`.
   - Sonuç: `docs/gyeon-products-dump.json` (~111 primary SKU).

2. **Section listesini hazırla**: `docs/gyeon-faqs.json`'dan section_name unique listesi (~100 product-name section + 8 non-product section).

3. **5 paralel Explore subagent** (her biri ~20 section + tam Gyeon product dump alır):
   - Görev: "Bu section_name listesinde her bir entry için, Gyeon product dump'ında en iyi eşleşen `primary_sku`'yu bul. Eşleşme yoksa null döndür. **Confidence: high/medium/low.** EVO/REDEFINED/+ suffix'lerine ve boy/varyantlara dikkat et."
   - Çıktı formatı: `[{section_name, primary_sku, product_name, barcode, confidence, notes}]`.
4. **Birleştir + manuel review**: Tüm subagent çıktılarını `docs/gyeon-section-sku-map.json`'a yaz. `confidence: "low"` veya `null` olanları konsola dök → kullanıcı onayı.

### Phase 2 — FAQ sınıflandırma (kod, LLM yok)

`scripts/build-gyeon-sku-map.ts` her article için sınıflandırır:
- **PRODUCT**: `section_name` map'te ve `primary_sku !== null` → `target_sku = primary_sku`.
- **BRAND/CATEGORY** (8 non-product section, sentinel mapping):

  | Category | Section | target_sku |
  |---|---|---|
  | General Brand Questions | Brand Questions | `_BRAND:gyeon:brand` |
  | General Brand Questions | Sales & Distribution Questions | `_BRAND:gyeon:distribution` |
  | General Brand Questions | Certified Detailer Questions | `_BRAND:gyeon:certified_detailer` |
  | General Brand Questions | Gyeon Experience Center Questions | `_BRAND:gyeon:training` |
  | Q² Collection | General Product Question | `_BRAND:gyeon:q2_general` |
  | Q²M Collection | General Product Question | `_BRAND:gyeon:q2m_general` |
- **SKIPPED**: section eşleşmedi VE non-product değil → `docs/gyeon-faqs-skipped.json`'a yaz, kullanıcıya raporla.

Ara çıktı: `output/staging/gyeon_faqs_en.csv` (`target_sku, question_en, answer_en`).

### Phase 3 — Türkçe çeviri (LLM, batch)

`scripts/translate-gyeon-faqs.ts`:
- Gemini 2.5 Flash veya Claude Haiku 4.5; prompt cache ile sistem talimatı tek seferlik.
- Batch: ~20 Q&A pair / call → ~38 çağrı.
- **Verbatim koruma kuralları** (Menzerna L.7 kuralları ile aynı):
  - Ürün adları: `Q² Mohs EVO`, `Q²M Bathe+`, `Q²R Gelcoat`, `IR lamp`
  - Teknik birimler: `9H`, `pH 2-11`, `25 °C`, `50 ml`, `25 000 km`
  - URL'ler: değiştirme
  - Detaylama (cila/pasta/kaplama) Menzerna FAQ'lerindeki terminolojiye uy
- Idempotency: input row'un SHA256'sını anahtarla cache'le (`output/staging/gyeon_translation_cache.json`).
- Boyut kontrolü: `sku.length + q.length + a.length < 3800` (Botpress 4KB row limit, `seed-category-faqs.ts:39-44`'tekiyle aynı).
- Çıktı: `output/csv/gyeon_faqs.csv` + audit `docs/gyeon-faqs-en-tr.jsonl`.

### Phase 4 — Seed (`scripts/seed-gyeon-faqs.ts`)

`seed-menzerna-brand-faqs.ts` (`scripts/seed-menzerna-brand-faqs.ts:1-63`) mirror'ı:
1. **Idempotency**:
   - `_BRAND:gyeon:*` mevcut satırları sil (regex filter, `deleteTableRows`).
   - Product-specific Gyeon FAQ'leri için: insert öncesi `(sku, question)` hash bazlı dedup — mevcut satırları topla, çakışanları skip.
2. **Insert**: 25 satır/batch (`createTableRows`).
3. **Verification** (script içinde):
   - `client.findTableRows({search: "Q² Mohs EVO ile One EVO farkı"})` → ilgili sku gelmeli.
   - `client.findTableRows({search: "Gyeon sipariş nasıl"})` → `_BRAND:gyeon:distribution` gelmeli.
   - Toplam Gyeon FAQ row count yazdır.

### Phase 5 — Conversation instruction güncelleme

`src/conversations/index.ts:274-275`'e satır ekle:
```
- **`_BRAND:gyeon:<category>`** → Gyeon marka rehberi (gyeon.zendesk.com resmi FAQ'si), "Gyeon'un önerisi şöyle..." gibi sun
```

### Phase 6 — (Opsiyonel, ileri commit) Cross-reference mining

Q²M Polish FAQ'leri Q²M Prep, BaldWipe vb. ürünlere referans veriyor. `commit 14cec6c` paterni ile `productRelationsTable.use_with`'e mine et. **Bu commit ayrı ve sonra.**

---

## Kritik dosyalar (modified / created)

**Yeni:**
- `scripts/build-gyeon-sku-map.ts`
- `scripts/translate-gyeon-faqs.ts`
- `scripts/seed-gyeon-faqs.ts`
- `docs/gyeon-section-sku-map.json` (subagent + manuel review çıktısı)
- `docs/gyeon-products-dump.json`
- `docs/gyeon-faqs-en-tr.jsonl`
- `docs/gyeon-faqs-skipped.json` (eşleşmeyenler raporu)
- `output/csv/gyeon_faqs.csv`

**Edit:**
- `src/conversations/index.ts` (~line 275, tek satır ekle)

**Reuse:**
- `scripts/seed-menzerna-brand-faqs.ts` — birebir pattern
- `scripts/seed-category-faqs.ts:39-44` — 4KB row size guard
- `src/tables/product-faq.ts` — schema referansı

---

## Verification (end-to-end)

1. `bun run scripts/build-gyeon-sku-map.ts` → JSON üretildi mi, `low/null` kayıt var mı?
2. **Manuel review** `docs/gyeon-section-sku-map.json` → kullanıcı onayı.
3. `bun run scripts/translate-gyeon-faqs.ts` → CSV üretildi, satır sayısı, encoding UTF-8.
4. 10 random çeviri spot-check (audit JSONL üzerinden).
5. `adk run scripts/seed-gyeon-faqs.ts` → cloud insert, "✓ inserted: N/M" çıktısı.
6. Semantic search testleri (script içinde 3 query, sonuç similarity > 0.7).
7. `adk evals` mevcut suite (Gyeon ile ilgili question varsa) — regression yok.
8. Manuel chat: `adk chat` ile "Q² Mohs EVO ile One EVO farkı?" ve "Gyeon nasıl distribütör olunur?" → doğru SKU ve _BRAND:gyeon FAQ retrieved.

---

## Bilinmeyenler / risk

- **Gyeon API rate limit**: Zendesk public API rate limit anonim için ~ saatte 200; Phase 1'de tek script çalışır, ürün dump zaten yerel JSON'dan, tekrar API çağrısı YOK.
- **Çeviri kalitesi**: bazı çift anlamlı terimler (e.g. "Light Box") TR'ye çevrilmemeli. Prompt'ta whitelist tut.
- **4KB row limit**: 752 FAQ'in büyük çoğunluğu küçük (≤500 char). Translation TR'de %20-40 büyüyebilir. Pre-flight check şart.
- **Eşleşmeyen sectionlar**: kullanıcı "atlarsın" dedi ama eşleşmeme oranı %5'ten yüksekse map'i revize et.
- **Idempotency sınırı**: Botpress `deleteTableRows` `_BRAND:gyeon:*` regex filter destekliyor mu? Menzerna scripti aynısını yaptığı için OK kabul edildi.
