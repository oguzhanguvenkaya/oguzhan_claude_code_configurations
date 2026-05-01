# Phase 1.1.10 — DB Veri Zenginleştirme + Anti-Halüsinasyon Sertleştirme (REV2)

## Revizyon Notu — 2026-04-30

İlk plan'da SKU → ürün adı tablosu DB'den doğrulanmamıştı; isimler eski belleğimden uydurulmuştu (örn. 26919.271.001 "Power Lock Ultra" değil **Ceramic Spray Sealant**, 700096 "TUGA Hydrocoat" değil **INNOVACAR SC1 SEALANT**, 74062 "Opti-Coat ProDrop" değil **FRA-BER Nanotech**, vb.). Bu plan SKU bazlı çalışır; ürün adları DB'den okunur. Ayrıca:
- `contains_sio2` şu an EAV'ye projection edilmiyor (`project-specs-to-meta.ts` SCALAR_KEYS'te yok)
- "5 → 8" iddiası yanlış: bugün `paint_protection_quick` için `product_meta.contains_sio2` satırı YOK; gerçek beklenti **0 → 8 true + 9 false**
- pH nötr stratejisi: `template_sub_type='ph_neutral_shampoo'` SSOT; numeric `ph_level` filter sadece kullanıcı açıkça "pH 7" / "6.5-7.5 arası" derse uygulanır. Bathe (sub_type ph_neutral, ph_level=6) bu sorgu için **dışlanmamalı**.

## Context

Phase 1.1.9 commit `bec827b` push'landı; çelişki temizliği işe yaradı. 4 trace iki kök neden gösterdi:
1. **Veri eksikliği**: `paint_protection_quick` 22 üründe `contains_sio2` boş; `ph_neutral_shampoo` 1 yanlış sınıflama (Neve = ön yıkama) + 2 string `ph_level` (Camper, Bersolux); Camper için DB'de specs.ph_level **null** görünüyor (verify gerekli).
2. **F2 anti-halüsinasyon kategori önerilerinde delik**: "GYEON Bathe genelde tercih edilir" gibi output dışı marka/ürün cümleleri (KQEJ0X).

İki hat ile kapatılır.

## Hat A — DB Veri Zenginleştirme

### A.0 — `project-specs-to-meta.ts` patch (ÖNKOŞUL)

[retrieval-service/scripts/project-specs-to-meta.ts](retrieval-service/scripts/project-specs-to-meta.ts):
- `SCALAR_KEYS` listesine `contains_sio2` ekle (boolean projection)
- `STALE_KEYS` listesine de `contains_sio2` ekle → idempotent re-project güvenli
- Re-project pipeline `value_boolean` kolonuna yazmalı (mevcut diğer boolean key'lerle aynı pattern)

**Bu eklenmeden A.1'in EAV etkisi sıfır olur.**

### A.1 — `contains_sio2` SKU bazlı set (8 true + 9 false)

Script: `retrieval-service/scripts/phase-1-1-10-data-fixes.ts`

Çalışma prensibi:
- Sadece SKU listesine güven; ürün adlarını DB'den `SELECT sku, name, brand, template_group, template_sub_type, specs FROM products WHERE sku = ANY($1)` ile oku
- Her SKU için before/after JSON yaz (`scripts/audit/phase-1-1-10-contains-sio2.json`)
  - `before`: `{sku, name, brand, current_contains_sio2, template_group}`
  - `after`: `{sku, new_value}`
- Beklenmedik bir SKU'da `template_group != 'paint_protection_quick'` ise WARN log + yine de update et (kullanıcı açık karar verdi — Q2-QV120M dahil)

**`contains_sio2 = true` (8 SKU, kullanıcı onayı):**
```
26919.271.001
700096
700097
74062
79304
Q2M-CDYA1000M
Q2M-WCYA4000M
Q2-QV120M
```

**`contains_sio2 = false` (9 SKU, kullanıcı onayı):**
```
22070.261.001
22870.261.001
70545
71331
74059
75182
78779
79301
Q2-W175G
```

**Not:** Plan tablosunda ürün adı yazmıyorum — DB'den audit script okuyacak. Önceki plan'daki adlar yanlıştı, bu plan SKU üzerinde çalışır.

### A.2 — `ph_neutral_shampoo` 3 fix (verify-first)

| SKU | Eylem | Verify before |
|---|---|---|
| 71490 (Neve, ph_level=10.35) | `template_sub_type = 'prewash_foaming_shampoo'` | DB'de mevcut sub_type'ı log'la |
| 70616 (Bersolux, specs.ph_level="Nötr") | `specs.ph_level = 7` (integer) | Önce string değeri audit JSON'a yaz |
| 701851 (Camper Wash) | `specs.ph_level` DB'de **null** görünüyor; eğer null ise → `7` set et + audit'e "kaynak yok, manuel karar" notu yaz; eğer "7-8" varsa → 7 set et | Mevcut değeri audit'e yaz |

### A.3 — Verification (kabul kriterleri)

```sql
-- 1) EAV projection çalıştı mı?
SELECT COUNT(*) FROM product_meta WHERE key='contains_sio2' AND value_boolean=true;
-- Beklenti: 8

SELECT COUNT(*) FROM product_meta WHERE key='contains_sio2' AND value_boolean=false;
-- Beklenti: 9

-- 2) pH nötr fix
SELECT sku, template_sub_type FROM products WHERE sku='71490';
-- Beklenti: prewash_foaming_shampoo

SELECT sku, specs->>'ph_level' FROM products WHERE sku IN ('70616','701851');
-- Beklenti: 7, 7 (integer string)
```

## Hat B — Bot Instruction Sertleştirme

### B.1 — F2 anti-halüsinasyon kategori-genel öneriye genişlet

[Botpress/detailagent-ms/src/conversations/index.ts](Botpress/detailagent-ms/src/conversations/index.ts) Adım 2 Madde 1 (Output sunum kuralları):

```diff
  - "X markası ürünleri" iddiası: SADECE productSummaries'taki TÜM brand field'ları aynıysa.
+ - Övgü/öne çıkarma cümleleri ("genelde X tercih edilir", "en çok kullanılan Y",
+   "öne çıkan Z", "öneririm Q", "özellikle W") → cümlede geçen ürün/marka adı,
+   bu turn'deki tool output'unda (productSummaries[].name + brand) yoksa YAZMA.
+   Output dışından isim hatırlamak halüsinasyon — yasak.
```

### B.2 — metaFilter agresifliği uyarısı (TOOL SEÇİMİ altı)

```diff
+ metaFilter agresifliği:
+ - templateSubType + numeric range filter çakışması: templateSubType varsa o
+   sub_type'ın tanım gereği garanti ettiği özelliği numeric metaFilter ile YENIDEN
+   filtreLEME — false-negative üretir.
+ - "pH nötr şampuan" → templateSubType='ph_neutral_shampoo' YETERLI.
+   ph_level 6.5-7.5 metaFilter EKLEME (Bathe ph=6 ama sub_type pH nötr — sub_type SSOT).
+ - ph_level numeric metaFilter SADECE kullanıcı açıkça "pH 7 olan", "6.5-7.5 arası",
+   "asidik şampuan", "alkali şampuan" gibi numeric/kategorik isteyince.
+ - contains_sio2 / contains_alcohol gibi binary filter 0 sonuç döndürürse: filter'ı
+   düşür, tekrar dene; veya "veri tabanında etiketli ürün az, semantik benzer
+   alternatifler" diliyle kategori düzeyinde sun (içerik-iddiası ETME).
```

**Toplam diff:** ~12 satır net (sadece ekleme).

## Kritik Dosyalar

- [retrieval-service/scripts/project-specs-to-meta.ts](retrieval-service/scripts/project-specs-to-meta.ts) — SCALAR_KEYS + STALE_KEYS patch
- [retrieval-service/scripts/](retrieval-service/scripts/) — yeni `phase-1-1-10-data-fixes.ts` (SKU bazlı, DB-read audit)
- [Botpress/detailagent-ms/src/conversations/index.ts](Botpress/detailagent-ms/src/conversations/index.ts) — Adım 2 Madde 1 + TOOL SEÇİMİ
- DB tabloları: `products`, `product_meta`

## Implementation Steps

### Step 1 — `project-specs-to-meta.ts` patch (~5 dk)
- `SCALAR_KEYS` ve `STALE_KEYS` içine `contains_sio2`
- Boolean projection branch'i mevcut bir boolean key'in pattern'ini takip etsin (örn. `is_featured` veya benzer)

### Step 2 — Audit + bulk update script (~25 dk)
`retrieval-service/scripts/phase-1-1-10-data-fixes.ts` oluştur:
- 17 SKU + 3 pH SKU için DB read (name, brand, current specs, sub_type)
- Audit JSON yaz: `scripts/audit/phase-1-1-10-{contains-sio2,ph-fix}.json`
- `UPDATE products SET specs = jsonb_set(specs, '{contains_sio2}', $value) WHERE sku=$sku`
- pH için: `UPDATE products SET specs = jsonb_set(specs, '{ph_level}', '7'::jsonb) WHERE sku IN ('70616','701851')`
- Neve için: `UPDATE products SET template_sub_type='prewash_foaming_shampoo' WHERE sku='71490'`

```bash
cd retrieval-service && bun run scripts/phase-1-1-10-data-fixes.ts
```

### Step 3 — EAV re-project (~5 dk)
```bash
bun run scripts/project-specs-to-meta.ts
```

A.3 verification SQL'leri çalıştır.

### Step 4 — Instruction edits (~10 dk)
B.1 + B.2 ekle.

### Step 5 — Build + commit + push (~5 dk)
```bash
cd Botpress/detailagent-ms && adk build
git add -A
git commit -m "fix(bot+db): Phase 1.1.10 — contains_sio2 EAV projection + 17 SKU enrichment + ph_neutral fix + F2 genişlet"
git push
```

**Fly deploy YOK** — DB Supabase'de anında, instruction bot tarafı.

## Verification / Test Plan (kabul kriterleri)

| # | Test | Beklenen | Hangi Fix |
|---|---|---|---|
| 1 | `SELECT COUNT(*) FROM product_meta WHERE key='contains_sio2' AND value_boolean=true` | 8 | A.0 + A.1 |
| 2 | `SELECT COUNT(*) FROM product_meta WHERE key='contains_sio2' AND value_boolean=false` | 9 | A.0 + A.1 |
| 3 | "seramik içerikli hızlı cila" / paint_protection_quick + sio2 | WetCoat, CeramicDetailer, Menzerna Ceramic Spray Sealant gibi 8 ürün — wax/cila ürünleri girmiyor | A.1 |
| 4 | "pH nötr şampuan öner" | **Bathe DAHIL**, Bersolux dahil, Camper dahil, Neve HARİÇ | A.2 + B.2 |
| 5 | "pH 7 olan şampuan" / "6.5-7.5 arası şampuan" | Bathe HARİÇ (numeric filter doğru — ayrı senaryo) | B.2 |
| 6 | KQEJ0X retest — "pH nötr şampuan, GYEON varsa söyle" output GYEON yoksa | "Sonuçlarda GYEON yok; FRA-BER ... öneriyorum" — "GYEON Bathe genelde tercih edilir" gibi cümle YOK | B.1 |
| 7 | `SELECT sku, template_sub_type FROM products WHERE sku='71490'` | `prewash_foaming_shampoo` | A.2 |
| 8 | `SELECT sku, specs->>'ph_level' FROM products WHERE sku IN ('70616','701851')` | "7", "7" | A.2 |

## Out of Scope

- Backend retrieval algoritma (`applyExactMatch` Fallback 2 bug — Phase 1.1.11+)
- Hardcoded sub_type/marka mapping
- Multi-turn eval otomasyonu (Phase 1.1.11)
- Eksik trace `conv_01KQDJYFC75V3V69RW6RQFN58E` analizi

## Phase 1.1.11 — Kapsamlı Multi-Turn Eval (sonraki)

20 senaryo (Set A–E: Anti-Halüsinasyon, Bütçe Kademeli, Variant Awareness, Belirsiz Sorgu, Multi-Turn Bağlam) Phase 1.1.10 commit sonrası çalıştırılır.

## Notes

- **SKU SSOT**: Plan'da ürün adı yok (önceki plan adları DB ile uyumsuzdu) — script DB'den okur
- **EAV önkoşulu**: `contains_sio2` SCALAR_KEYS patch'i olmadan A.1 etkisiz
- **pH SSOT**: `template_sub_type` katalog dilinde nötr → numeric `ph_level` filter sadece numerik istek
- Veri zenginleşmesi false-negative'i azaltır → bot daha az "bulamadım" der → halüsinasyon basıncı düşer
- F2 genişlemesi training-data sızıntısını önler
