# Real-World Test Hatası Analizi + Veri/Filter Fix Paketi

## Context

Kullanıcı 20 gerçek senaryoyu manuel test etti. 7 problem raporladı:

- **Soru 4 (portakal kabuğu)** — alakasız cevap
- **Soru 6 (PPF bakım)** — parfüm, saklama kutusu carousel'de
- **Soru 7 (Mat PPF bakım)** — Foam Gun, PPF Renew (agresif pasta) önerildi; Cure Matte önerilmedi
- **Soru 8 (cam kireç)** — uygulama ipucu alakasız
- **Soru 14 (Bosch köpük)** — 2. turda FAQ kopyalama hissi
- **Soru 16 (3000 TL polisaj)** — carousel'de "Polish Aroma" (yok aslında); template_group yanlış şüphesi
- **Soru 17 (Tesla)** — degreasing ürünü carousel'de
- **Soru 19 (iç mekan MAT)** — "Dory" (marin tekne tik temizleyici), "all aliment" carousel'de

Trace analizi ile kanıtlı kök sebepler:

## Kök Sebepler (Trace-Verified)

### R1 — Veri sınıflandırma hataları (EN KRİTİK)

`productsMasterTable` 4+ satırda yanlış `template_group`:

| SKU | Ürün | Hata | Doğru |
|-----|------|------|-------|
| 75132 | FRA-BER Dory Marin Tekne Tik Temizleyici | `interior_cleaner / wood_cleaner`, main_cat=**(boş)** | `marin_products`, main_cat=`MARİN` |
| 77192 | FRA-BER X-Wood Marin Tekne Ahşap Koruyucu | `interior_cleaner / wood_protector`, main_cat=**(boş)** | `marin_products`, main_cat=`MARİN` |
| Q2M-DT10P | GYEON Degrease Tabs | `interior_cleaner / degreaser`, main_cat=`DIŞ YÜZEY` | main_cat ile group çelişkili |
| Q2M-PPR1000M | GYEON PPF Renew (agresif pasta) | `abrasive_polish / heavy_cut_compound` | PPF bakım aramasına düşer, YANLIŞ |

Marin ürünleri iç-mekan/interior_cleaner araması yaptığında CAROUSEL'e sızıyor (Dory, X-Wood).

### R2 — LLM filter seçim hataları

searchProducts'u sıklıkla FILTER'SIZ çağırıyor ([trace örnek](Botpress/detailagent/.adk/bot/traces/traces.db)):

```
Soru 6 (PPF bakım): query="PPF kaplama koruma ve bakım" (NO filter)
  → FRA-BER Bean Classic Araç Parfümü ❌
  → SGCB Saklama Kutusu ❌
  
Soru 7 (Mat PPF):   query="Matte PPF bakımı koruma" (NO filter)
  → SGCB Foam Gun ❌ (köpük tabancası, mat bakımla alakası yok)
  → GYEON PPF Renew ❌ (AGRESİF pasta — mat PPF'e asla uygulanmaz)
```

Template_group doğru seçilse bile PPF için özel karmaşık: `ppf_tools` (kurulum), `car_shampoo/ppf_shampoo`, `ceramic_coating/ppf_coating`, `matte_coating`. LLM hangi sub_type seçeceğini bilmiyor.

### R3 — searchFaq limit=1 hilesi

LLM bazen `searchFaq({limit: 1, query: ...})` çağırıyor. Tool confidence='low' dönerse bile tek kötü FAQ'yı kopyalıyor.

Örnek (Soru 14 Bosch): searchFaq limit=1 → random FAQ (3236NBR2 sünger kullanım talimatı) → bot metinde kullandı.

### R4 — LLM sunum kontrolü yok

Tool `carouselItems[]` dönüyor. Bot hepsini yield ediyor. Ürün adında "Marin", "Tekne", "PPF Installer" geçse bile, kullanıcı "araç iç mekan" veya "mat PPF bakım" istedi olsa bile filtrelemeden sunuyor.

## Fix Paketi — P5 → P8 (küçük, test-driven)

### P5 — Veri Sınıflandırma Düzeltmeleri (15 dk, veri)

**Dosya:** `Scripts/fix_data_misclassifications.py` (yeni)

Düzeltmeler:

| SKU | Alan | Yeni Değer |
|-----|------|-----------|
| 75132 (Dory) | template_group | `marin_products` |
| 75132 | template_sub_type | `wood_cleaner` (kalır) |
| 75132 | main_cat | `MARİN` |
| 77192 (X-Wood) | template_group | `marin_products` |
| 77192 | main_cat | `MARİN` |
| Q2M-DT10P | template_group | `interior_cleaner` (kalsın) |
| Q2M-DT10P | main_cat | **`İÇ YÜZEY`** (DIŞ YÜZEY değil) |

Uygula: CSV + Cloud upsert (`upsert-data-fixes.ts`).

**Test:** Soru 19 (iç mekan MAT) rerun → Dory, X-Wood carousel'de olmamalı.

### P6 — searchProducts Category Sanity Filter (10 dk, kod)

**Dosya:** [`src/tools/search-products.ts`](Botpress/detailagent/src/tools/search-products.ts)

Post-fetch filter: `templateGroup='interior_cleaner'` veya semantic "iç" keyword aramasında **main_cat='MARİN' olan ürünleri DIŞLA**. Backend sanity.

```ts
// Örnek — handler'da fetchResult.rows'a uygulanır
const isInteriorQuery = templateGroup === 'interior_cleaner' ||
  /iç\s+(mekan|temizl|yuze)/i.test(query || '');
if (isInteriorQuery) {
  fetchResult.rows = fetchResult.rows.filter(
    (r) => (r.main_cat as string) !== 'MARİN'
  );
}
```

**Test:** Soru 19 regresyon + Soru 13 (kumaş koltuk) korumalı.

### P7 — searchFaq Min-Limit Enforcement (5 dk, kod)

**Dosya:** [`src/tools/search-faq.ts`](Botpress/detailagent/src/tools/search-faq.ts)

Handler başında: `const effectiveLimit = Math.max(limit ?? 5, 3);`

LLM `limit=1` geçse bile min 3 FAQ döndür. Kötü tek-seçim riski azalır.

**Test:** Soru 14 (Bosch köpük) 2. turdaki FAQ kopyalama davranışı yumuşar.

### P8 — PPF Bakım Kategori Rehberliği (10 dk, instruction)

**Dosya:** [`src/conversations/index.ts`](Botpress/detailagent/src/conversations/index.ts) TOOL SEÇİMİ bölümü

PPF alt kategorilerini net tanımla:

```
PPF kaplı araç SORUSU → templateGroup seçimi:
- Rutin yıkama/bakım → 'car_shampoo' (PPF Wash önerilir)
- Koruma/seramik üstü kaplama → 'ceramic_coating', sub='ppf_coating' (Q2-PPFE50M)
- Mat PPF bakım → 'ceramic_coating', sub='matte_coating' (Q2-MTEL50M) + PPF Wash
- ASLA KULLANMA: 'ppf_tools' (kurulum aletleri, bakım değil)
- ASLA KULLANMA: 'abrasive_polish' (PPF Renew AGRESİF, hasar için — rutin bakımda yanlış)
```

**Test:** Soru 6 + Soru 7 rerun → PPF Wash, Matte EVO, Cure Matte gibi doğru ürünler.

## Sıralı Uygulama

```
P5 (veri fix) → Soru 19 rerun — Dory/X-Wood carousel'de olmamalı
   ↓
P6 (main_cat guard) → Soru 19 backup koruma; regresyon Soru 13
   ↓
P7 (faq min) → Soru 14 rerun — FAQ kopyalama azalır
   ↓
P8 (PPF rehberliği) → Soru 6+7 rerun — doğru PPF ürünleri
   ↓
Full real20 rerun → 20 soruda değişim ölç
```

## Kritik Dosyalar

- `Scripts/fix_data_misclassifications.py` — P5 CSV + SQL
- `Botpress/detailagent/scripts/upsert-data-fixes.ts` — P5 Cloud upsert
- [`Botpress/detailagent/src/tools/search-products.ts`](Botpress/detailagent/src/tools/search-products.ts) — P6 sanity filter
- [`Botpress/detailagent/src/tools/search-faq.ts`](Botpress/detailagent/src/tools/search-faq.ts) — P7 min limit
- [`Botpress/detailagent/src/conversations/index.ts`](Botpress/detailagent/src/conversations/index.ts) — P8 PPF rehberliği

## Riskler

| Risk | Seviye | Mitigation |
|------|:---:|-----------|
| P5 veri fix başka ürünlerde beklenmedik etkileme | Düşük | Sadece 3 SKU'yu etkiliyor, ismen spesifik |
| P6 main_cat guard meşru marin aramasını da keser | Orta | Sadece interior_cleaner VE "iç" keyword'ünde uygulanır |
| P7 min=3 FAQ gereksiz veri döner | Düşük | 3 satır küçük, tool output kabul edilebilir |
| P8 instruction şişmesi | Düşük | ~15 satır, LLM kuralı izleyebilir |

## Doğrulama (her P sonrası)

- **P5**: `adk run scripts/verify-data-fixes.ts` — Dory, X-Wood, Degrease Tabs doğru kategoride
- **P6**: `adk evals real20-19-parlak-ic-mekan` — carousel'de marin ürünü yok
- **P7**: `adk evals real20-14-portfoy-sampuan-vs-yikama` — 2. tur daha akıllı cevap
- **P8**: `adk evals real20-06-ppf-bakim real20-07-mat-ppf-bakim` — doğru PPF ürünleri

## Kapsam Dışı (Şimdilik)

- **Soru 4** (portakal kabuğu) — tool çağrıları doğru (heavy/finish polisaj), LLM sunumu problemli. Instruction tweak gerekebilir ama test-driven önceliği P5-P8'den düşük.
- **Soru 8** (cam kireç) — template_group iki denemeyle (glass_cleaner_protectant → glass_cleaner) doğru bulmuş. "Uygulama ipucu mantıklı değil" subjektif; veri kontrol gerekli ama hızlı fix değil.
- **Soru 16** ("Polish Aroma" screenshot'ta görülen ürün) — master'da YOK, hallucination olabilir. Tekrar edilebilir mi belli değil.
- **Soru 17** (Tesla degreasing) — clarifying question sonrası. Data fix (P5 Degrease Tabs main_cat) kısmen çözer.

Bu 4 soru P5-P8 sonrası rerun ile yeniden değerlendirilecek.

## Commit Sıralaması

- `fix(data): correct marin products misclassification (P5)`
- `fix(search): guard interior queries against marin products (P6)`
- `fix(faq): enforce min 3 results even when LLM requests 1 (P7)`
- `fix(instruction): PPF category routing guardrails (P8)`
