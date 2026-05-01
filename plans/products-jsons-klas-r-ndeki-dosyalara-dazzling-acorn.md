# Veri kalitesi audit — taxonomy + specs + FAQ

**Tarih:** 2026-04-23
**Veri kaynağı:** Live retrieval-service `/admin/*` (511 ürün · 165 sub_type · 3 156 FAQ)
**Soru:** 9 başlıklı kapsamlı veri sağlığı sorusu (capacity duplication, sub_type ekonomisi, schema tutarlılığı, anahtar isim dili, taxonomy yeterliliği, birleştirilebilir sub_type'lar, duplicate meta + FAQ)

---

## 1. capacity_total_lt vs capacity_ml vs capacity_liters — duplicate mi?

**Cevap karışık. Üç kavram farklı, ama implementation tutarsız.**

### sprayers_bottles · pump_sprayer (örnek SKU 26005, 19976, 20102)

```
26005   capacity_liters=1.8   capacity_total_lt=1.86  capacity_usable_lt=1.5
19976   capacity_liters=null  capacity_total_lt=2.0   capacity_usable_lt=1.8
20102   capacity_liters=null  capacity_total_lt=2.0   capacity_usable_lt=1.8
```

→ Bunlar **3 ayrı kavram**:
- `capacity_total_lt` = tankın fiziksel toplam hacmi (1.86 L)
- `capacity_usable_lt` = güvenlik çizgisi altında doldurabileceğin (1.5 L)
- `capacity_liters` = pazarlama/nominal değer (1.8 L; aslında "1.8 L" olarak satılır)

**Sorun:** 9 üründen 3'ünde `capacity_liters` null bırakılmış. Ya hep doldurulmalı ya hep `(usable + total)/2` gibi türetilmiş bir kural çıkarılıp drop edilmeli.

### sprayers_bottles · trigger_sprayer

```
24952        capacity_ml=600    capacity_liters=null
12922        capacity_ml=1035   capacity_liters=null
79587-643    capacity_ml=1000   capacity_liters=1.0   ← gerçek duplicate
```

→ Burada **gerçek duplicate var**: `capacity_ml=1000` ile `capacity_liters=1.0` aynı veriyi farklı ünitede saklıyor. Genel pattern olarak trigger'lar `capacity_ml` kullanıyor (1000 ml bazlı), pump'lar `capacity_liters` (büyük tank). Karışım gereksiz.

### Tüm katalog — capacity ailesi global coverage

| Key | Coverage | Aslında ne |
|---|---|---|
| `capacity_liters` | %10.2 | nominal/marketing |
| `volume_ml` | %8.0 | başka grup için (şampuan vs) |
| `capacity_total_lt` | %5.5 | sadece pump_sprayer |
| `capacity_usable_lt` | %5.1 | sadece pump_sprayer |

**Öneri:**
- Sprayer'lar: **tek standard `capacity_ml` (number)**. Nominal değer için `capacity_label` (string: "1.8 L"). usable + total ekstra alanlar olarak kalsın.
- Şampuan / kaplama / sıvı genel: **`volume_ml`** (number).
- `capacity_liters` deprecate, `volume_liters` deprecate.

---

## 2. Ortalama bir sub_type altında kaç ürün olmalı?

### Şu anki dağılım (165 sub_type · 511 ürün)

| Aralık | Sub_type sayısı | % |
|---|---|---|
| **1 ürün** | 82 | %50 |
| **2 ürün** | 27 | %16 |
| 3-5 ürün | 38 | %23 |
| 6-15 ürün | 15 | %9 |
| 16+ ürün | 3 | %2 |

→ **Ortalama 3.1 / median 2** — vahşi şekilde fragmente. **Sub_type'ların yarısı (82 adet) tek ürünlü.**

### Pratik hedef

**Anlamlı bir sub_type için min 4-5 ürün** olmalı:
- Daha az ürün = arama filter'i yararsız (ürün zaten 1, kullanıcı doğrudan SKU/ad'la bulur)
- Bot "X kategorisinde ürün öner" deyince en az 3 alternatif gösterebilmeli
- Faceted search için statistically meaningful aggregation yapabilmek için min ~5 ürün

### Yorum

**1-2 ürünlü 109 sub_type → bunların %80'i parent gruba veya yakın sub_type'a merge edilebilir.** Geriye ~50-60 anlamlı sub_type kalır. Bu Phase 6.5 enrichment'ın asıl iş kalemlerinden biri.

---

## 3. Aynı sub_type'ta tüm meta'lar aynı olmalı mı? Null OK mu?

**Cevap:** Schema'nın **stable bir base'i** olmalı, kalan alanlar opsiyonel.

### Mevcut schema consistency (örnek tarama, 5-10 ürün/sub_type)

| Sub_type | Universal key (n/n) | Ortak ≥%60 | Nadir <%30 |
|---|---|---|---|
| pump_sprayer | 10/24 | 9 | 3 |
| trigger_sprayer | 6/34 | 10 | 7 |
| **prewash_foaming_shampoo** | **3/52** | 14 | **28** |
| paint_coating | 4/19 | 8 | 5 |
| heavy_cut_compound | 3/27 | 13 | 7 |
| foam_pad | 9/19 | 4 | 5 |
| vent_clip | 6/15 | 1 | 7 |
| ph_neutral_shampoo | 4/35 | 14 | 13 |

### Yorum

- **Tutarlı sub_type'lar** (foam_pad, pump_sprayer): 9/19 — 9/24 universal. Yarısı her üründe. Sağlıklı.
- **Tutarsız sub_type'lar** (prewash_foaming_shampoo: 3/52, ph_neutral_shampoo: 4/35): aynı sub_type'ta 52 farklı key var, sadece 3'ü her üründe — schema'nın disipline edilmesi gerekiyor.

### Doğru pattern

```
Universal:                 anlatı triple (howToUse + whenToUse + whyThisProduct) — %100
Sub_type schema base:      ~5-10 key, sub_type identity'sinin parçası — %100 zorunlu
Sub_type opsiyonel:        ~3-7 key, ürün-spesifik özellikler — null OK ama anlamlı
Marka-spesifik niş:        1-2 üründe görülen — kabul edilebilir, ama not olarak markdown'a düşmeli
```

**Null OK mı?** Evet — ama ikiye ayır:
- `null` = bilinmiyor / henüz veri girilmedi → **doldurulmalı**
- `null` = anlamlı değil (ör. fragrance'da `durability_months` doldurulmaz) → **schema'dan o sub_type için silinmeli**

İkisi karışıyor şu an. UI'da "varsayılan: null" ile "uygulanamaz" arasında bir flag yok.

---

## 4. Botpress Tables bağı yok artık — istediğimiz schema'yı yapabilir miyiz?

**Evet, tam olarak.**

- Bot artık microservice'in `/search`, `/products/:sku`, `/faq` endpoint'lerinden **mirror response contract**'a göre veri alıyor (Phase 4 cutover).
- Bot tarafı raw `products.specs` JSONB görmüyor — formatter `productSummaries[]` şeklinde **sunum-için dönüştürülmüş** veri yolluyor.
- Yani **DB schema'sını kırmadan tüm sub_type'ları + specs key'lerini yeniden adlandırabiliriz**. Sadece formatter ([retrieval-service/src/lib/formatters.ts](retrieval-service/src/lib/formatters.ts)) ve slot extractor'ı ([retrieval-service/src/lib/slotExtractor.ts](retrieval-service/src/lib/slotExtractor.ts)) güncellemek yeter.

**Bağımlılık matrisi:**
| Değişim | Etkilenen taraf | Eylem |
|---|---|---|
| `template_group` rename | sub_type pattern map | slotExtractor güncelle |
| `template_sub_type` consolidation | Bulk wizard zaten var | Çalıştır |
| `specs.X` → `specs.Y` | Formatter label map + meta filter resolver | İki dosya |
| Yeni `specs.foo` ekleme | Frontend SpecEditor jenerik render eder | Otomatik |
| FAQ schema | Bot template'i değişmez | — |

---

## 5. İngilizce key isimleri + Türkçe chatbot — sorun mu?

**Hayır.** Ama bir koşulla.

### Neden sorun değil
- Bot LLM (Gemini) **raw key isimlerini görmez** — formatter'lar `productSummaries[]` içine zaten Türkçe label'larla dönüştürür ("ph_level" → "pH değeri" gibi).
- LLM'in tool description'larındaki örnekler de Türkçe yazılmış (Phase 4 instruction).
- Postgres tarafı için camelCase/snake_case key'ler zaten standart, çoklu dil sorgularında stable.
- Türkçe key (`fiyat`, `dayanım_ay`) yapsaydık SQL ı↔i sorunu, JSONB path quoting karmaşası gelirdi.

### Tek koşul
Frontend / admin UI'da kullanıcıya gösterilen yerlerde **label translation map** olmalı. Şu an:
- `/products/[sku]` Specs tab — raw key gösteriliyor (`ph_level`, `durability_months`). Geliştirici-friendly ama operatör için ikinci satır çevirisi olabilir.
- Bot tarafında zaten formatter Türkçeye çeviriyor (carousel title vb.).

**Öneri:** `lib/data/spec-labels.ts` gibi bir map (`{ph_level: "pH değeri", durability_months: "Dayanım (ay)"}`) tek bir yerde tut. Operatör UI'sında label, code'da key.

---

## 6. Daha fazla template_group gerekli mi?

**Hayır — tam tersi, mevcut 26 yeterli, hatta birkaçı bile birleşebilir.**

### Mevcut 26 grup — ürün dağılımı

```
fragrance(93) sprayers_bottles(48) polishing_pad(33) microfiber(31) 
car_shampoo(30) spare_part(28) interior_cleaner(25) abrasive_polish(24) 
ceramic_coating(23) polisher_machine(23) storage_accessories(23)
paint_protection_quick(22) contaminant_solvers(21)
applicators(14) ppf_tools(14) industrial_products(12)
clay_products(8) tire_care(7) leather_care(7)
brushes(6) marin_products(5) glass_cleaner_protectant(5)
masking_tapes(4) glass_cleaner(2) product_sets(2) accessory(1)
```

### Sorun template_group sayısında değil — sub_type granülasyonunda

26 grup zaten yüksek granülarite (Shopify catalog'larında genelde 5-12 üst kategori). Ürün başına ortalama 19.7 — sağlıklı. Sorun:

- 165 sub_type / 511 ürün = 3.1 — **çok dar** (sub_type'ların yarısı 1 ürünlü)
- glass_cleaner (2 ürün) ile glass_cleaner_protectant (5 ürün) ayrı groups → **birleştirilebilir** (tek glass_care)
- product_sets (2 ürün) + accessory (1) → muhtemelen başka grupların içine eritilir

### Belki eklenebilecek tek grup
**fragrance** çok kalabalık (93 ürün, %18) — `fragrance` kalsın ama içinde sub_type'lar (vent_clip 55, spray_perfume 10, hanging_card 9, ...) zaten ayrım yapıyor. Yeni grup gerekmiyor.

---

## 7. Birleştirilebilir sub_type adayları (P0/P1/P2)

### P0 — Kesin merge (1-2 ürünlü, anlamı çakışan)

| Grup | Mevcut | → Hedef |
|---|---|---|
| ceramic_coating | paint_coating_kit + multi_step_coating_kit + single_layer_coating | → **paint_coating** |
| ceramic_coating | matte_coating, tire_coating, wheel_coating, trim_coating, leather_coating, ppf_coating, top_coat, spray_coating, interior_coating, fabric_coating | → 3 ana sub'a daralt: **paint_coating, glass_coating, surface_coating** (diğerleri specs.application_surface'a düşer) |
| spare_part | maintenance_kit + nozzle_kit + extension_kit | → **maintenance_kit** |
| spare_part | nozzle + nozzle_kit | → **nozzle** |
| contaminant_solvers | iron_remover + wheel_iron_remover | → **iron_remover** (specs.application_area="wheel") |
| sprayers_bottles | foaming_pump_sprayer + pump_sprayer | → **pump_sprayer** (specs.foaming=true) |
| leather_care | leather_care_kit + leather_conditioner + leather_protectant | → **leather_kit** (3 ayrı sub_type'ta toplam 5 ürün) |

### P1 — Önerilen merge (microfiber bez kategorileri)

`microfiber` grubunda 12 sub_type var, ama **çoklu çakışıyor**:
- multi_purpose_cloth ↔ buffing_cloth ↔ glass_cloth ↔ interior_cloth ↔ cleaning_cloth ↔ suede_cloth ↔ coating_cloth

→ Önerilen tek pattern: **`general_cloth`** (multi-purpose) + **`specialty_cloth`** (glass/coating/suede özel kullanım için). Sub_type 12'den 4-5'e iner.

### P2 — Veri zenginleştikçe (ürün artarsa)

interior_cleaner'daki 16 sub_type'ın birçoğu (wood_cleaner, wood_protector, foam_cleaner, plastic_restorer) tek ürünlü. Eğer kategori büyümeyecekse (8+ ürün hedefi), bunlar `specialty_interior_cleaner` çatısı altında toplanır.

### Rakam

Aşırı agresif: 165 sub → ~70 sub.
Konservatif (sadece P0): 165 sub → ~140 sub.
**Öneri: P0 + P1 = ~110 sub** (median 4-5 ürün/sub).

---

## 8. Duplicate meta key aileleri

Global coverage'tan tespit edilen **6 ana duplicate ailesi**:

| Aile | Çakışan keyler | Öneri |
|---|---|---|
| **volume / capacity** | capacity_liters, capacity_ml, capacity_total_lt, capacity_usable_lt, volume_ml, volume_liters, volume_kg | → 2 standard: `volume_ml` (sıvı), `capacity_ml` (sprayer tank). Diğerleri deprecate. |
| **durability** | durability_months, durability_days, durability_km, durability_label, durability_weeks, durability_washes | → `durability_months` (number) + `durability_km` (number, opsiyonel). Diğerleri label'a indir. |
| **ph** | ph, ph_level, ph_label, ph_tolerance | → `ph_level` (number, şampuan), `ph_tolerance` (string range "2-11", kaplama). 2'sine indir. |
| **dilution** | dilution, dilution_ratio, dilution_scale, dilution_foam_lance, dilution_kova, dilution_pump, dilution_manual, dilution_steps_ml | → `dilution_ratio` (string "1:10") + `dilution_methods` (object: {bucket:"1:200", foam_lance:"1:50"}) — sub-key'ler iç içe yapılandır. |
| **consumption / coverage** | consumption_ml_per_car, consumption_ml_per_cabin, coverage_ml_per_sqm, recommended_bucket_ml, recommended_foam_cannon_ratio | → `consumption_ml_per_car` (number) + `application_method_notes` (text) |
| **safe_on / compatibility** | safe_on_ceramic_coatings, safe_on_ppf_wrap, aluminum_safe, fiberglass_safe, plexiglass_safe | → `compatibility` (array of strings: ["ceramic_coating","ppf"]) yerine boolean salgını |

### features vs sub_type

Specs içinde hem `features` (array, %33 coverage) hem `sub_type` (string, %32 coverage) var — ama `template_sub_type` zaten ayrı kolon. **Specs içindeki `sub_type` field redundant — silinebilir.** `features` kalsın.

---

## 9. Duplicate FAQ soruları

### Birebir aynı soru sayısı: **323**

```
75×  "Bu ürün nedir?"
72×  "Koku ne kadar süre kalıcıdır?"
59×  "Açıldıktan sonra ürün nasıl saklanır? Raf ömrü ne kadardır?"
56×  "Nereye uygulanır?"
38×  "Nasıl uygulanır?"
37×  "Nasıl kullanılır?"
31×  "Birden fazla kat uygulayabilir miyim?"
25×  "Ne kadar dayanır?"
21×  "Bu ürünle kaplı bir araç nasıl bakım görür?"
21×  "Çıkarılabilir mi?"
```

### İki yorum

**(A) "Aynı soru, farklı cevap" pattern (template FAQ):**
"Bu ürün nedir?" 75 üründe — her birinde **farklı cevap**. Bu bir **FAQ şablon politikası**: bot search index'inde bu kalıpları aramak (örn. "ne işe yarıyor" → her ürün için kendi cevabı) için bilinçli bir tasarım. Kaldırılırsa bot generic "Bu ürün nedir?" sorusunda product-specific cevap üretemez (full_description'a düşer; ama o bazen daha hantal).

**Karar:** Bu duplicates **hata değil** — kasıtlı template FAQ'lar. Bot tarafında SKU bypass ile çalışıyor (Phase 4 searchFaq).

**(B) Gerçek duplicate olabilir:**
- "Nasıl uygulanır?" (38) vs "Nasıl kullanılır?" (37) → bunlar **aynı semantik** ama farklı string. Vector search ikisini ayırt edemez. **Bu gerçek dup.** Birleştirilmeli.
- "Bu ürünle kaplı bir araç nasıl bakım görür?" (21) — sadece kaplama ürünleri için, şablon. OK.
- "Üzerine ek kaplama uygulayabilir miyim?" (13) + "Üzerine ek kaplama/wax/sealant uygulayabilir miyim?" (11) → **near-duplicate**, normalize edilmeli.

### Pratik aksiyon

| Aksiyon | Etkilenen FAQ |
|---|---|
| "Nasıl uygulanır?" + "Nasıl kullanılır?" tek soruya birleştir (cevap concat veya birini sil) | ~75 |
| "Üzerine ek kaplama..." varyantları normalize et | ~24 |
| Pattern soruları **template_faqs** tablosuna taşı (1× soru, ürün başına bağ) | 320+ → 30 satır |

Üçüncü öneri en güçlü olanı: 3 156 FAQ → ~600 unique soru kalır, geri kalanı template instance.

---

## Özet aksiyon matrisi

| # | Sorun | Etki | Aksiyon | Sırası |
|---|---|---|---|---|
| 1 | capacity_* karışıklığı | Filter güvenliği | volume_ml + capacity_ml standart | P0 |
| 2 | 109 sub_type 1-2 ürün | Search verimsiz | P0+P1 merge → ~110 sub | P0 |
| 3 | Sub_type içi schema tutarsız | LLM güvensiz | Sub_type schema base tanımla, eksikleri null'la veya doldur | P1 |
| 4 | 6 duplicate key ailesi | Filter veri kaybı | Specs key normalize wizard (var) — her aile için bir koşu | P0 |
| 5 | features vs sub_type vs template_sub_type | Yapı kirliliği | specs.sub_type drop | P1 |
| 6 | "Nasıl uygulanır" + "Nasıl kullanılır" dup | RAG karışık match | FAQ merge | P1 |
| 7 | 323 template FAQ duplicate | Index şişkin (ama by design) | template_faqs tablosu — opsiyonel | P2 |
| 8 | Daha fazla template_group | — | **Gereksiz, mevcut 26 yeterli** | — |
| 9 | English key + Turkish bot | — | **Sorun değil, label map zaten formatter'da** | — |

---

---

## 10. "25 kg şampuan" gibi sayısal sorgular — meta key azaltma SKU bulmayı zorlaştırır mı?

**Cevap: HAYIR — şart bir şey: meta key isimleri standartlaşır, veri kalır.**

### Yanlış anlaşılma riski
"Meta key azaltma" demek **veri silmek değil**. Aşağıdaki ikisi farklı:

| ❌ Bu DEĞİL | ✅ Bu |
|---|---|
| 25 kg değerini sil | 25 kg değerini tek bir canonical key altına taşı |
| Şampuanın volume bilgisini DB'den at | `volume_kg`, `volume_liters`, `capacity_kg`, `volume`, `boyut` → **`volume_kg`** standardı |
| product_meta tablosunu boşalt | product_meta tablosunda 5 farklı isimle değil tek isimle ara |

### Mevcut altyapı — 25 kg şampuan sorgusu nasıl çözülür

**Kod akışı (Phase 4 cutover sonrası, şu an çalışıyor):**

1. **Bot LLM intent parse:**
   ```
   user: "25 kg şampuan ister misin"
   bot: searchProducts({
     query: "şampuan",
     templateGroup: "car_shampoo",
     metaFilters: [{ key: "volume_kg", op: "eq", value: 25 }]
   })
   ```
   `metaFilters` parametresi searchProducts tool'unun parçası — [Botpress/detailagent-ms/src/tools/search-products.ts:142-180](Botpress/detailagent-ms/src/tools/search-products.ts#L142-L180). LLM tool description'da örnekleri görüyor.

2. **Microservice resolveMetaFilterSkus:**
   ```sql
   SELECT sku FROM product_meta
   WHERE key = 'volume_kg' AND value_numeric = 25
   ```
   [retrieval-service/src/lib/searchCore.ts:81-110](retrieval-service/src/lib/searchCore.ts#L81-L110). Eşleşen SKU set'i döner. AND semantics (birden fazla filter kesişir).

3. **Hybrid search içinde intersect:**
   Vector + BM25 araması yapılır, sonuçlar meta SKU set'ine intersect edilir. Sonuçta sadece `volume_kg=25` olan + "şampuan" semantic match'i.

### Vector search 25 kg'ı tek başına bulabilir mi?

**Hayır** — ve bu meta key azaltmayla ilgili değil, vektör semantiğinin doğasıyla ilgili:

- Vector search SEMANTIC similarity → "şampuan" / "yıkama maddesi" / "ön yıkama köpüğü" arası yakınlık.
- "25" sayısı embedding uzayında **çok zayıf sinyal** (rakam token'ı, contextual değil).
- Aynı vektör 25 kg, 5 litre, 250 ml şampuanları yakın yakın getirir.

**Numeric attribute araması = SQL filter işi**, vector değil. Bu yüzden microservice **hybrid pipeline** — vector yarısı semantic, SQL yarısı exact. İkisi birlikte çalışır.

### Şu anki engeller (ölçüldü)

| Engel | Açıklama | Çözüm |
|---|---|---|
| `volume_kg` muhtemelen henüz product_meta'da yok | Ürünlerin çoğunda volume bilgisi `specs.capacity_liters` veya `specs.volume_ml` JSONB içinde, ama meta tablosuna projekte edilmemiş. resolveMetaFilterSkus sadece product_meta'ya bakıyor. | **Migration / seed**: specs içindeki volume değerlerini product_meta'ya da yaz. Veya specs JSONB üzerinden de filter yapan ek path. |
| Çoklu unit (kg, lt, ml) | 25 kg ↔ 25 L şampuan ↔ 25 000 ml — aynı paket birim farklı | Canonical olarak `volume_ml` (number) tut, 25 kg → 25000 ml seed'le. veya `volume_unit` ek key ile birim ayır. |
| LLM hangi key'i seçer? | `volume_kg` mi `volume_ml` mi? Tool description'da örnek olmazsa yanlış key dener | searchProducts metaFilters describe'ında **canonical key listesi + örnekler** ekle (şu an birkaç örnek var ama eksik) |

### Sub_type azaltma SKU bulmayı **zorlaştırmaz, kolaylaştırır**

| Senaryo | 165 sub_type ile | 110 sub_type ile (P0+P1 merge sonrası) |
|---|---|---|
| "boya seramik kaplama" | LLM şu 4'ten birini seçmeli: paint_coating, paint_coating_kit, multi_step_coating_kit, single_layer_coating. Yanlış seçerse 0 sonuç. Şu an `expandSubTypeFamily()` patch ile kompanse ediliyor. | LLM tek seçenek `paint_coating`. Patch gereksiz. Doğru filter %100. |
| "iron remover" | iron_remover (2) + wheel_iron_remover (4) ayrı. Kullanıcı "demir tozu sökücü" dese hangisini seçecek? Eşit semantic. | Tek `iron_remover` + specs.application_area="wheel" boolean. Filter dağılmaz. |
| "1 ürünlü sub_type" (örn. wood_protector) | LLM templateSubType="wood_protector" yazdı → 1 ürün döner. Ama bu kullanıcı için "öneri" sayılmaz, faceted listede yararsız. | Sub_type kalkar, ürün `interior_cleaner` altında specs.material="wood" filter ile bulunur. Daha temiz. |

→ **Sub_type azaltma = sinyal güçlenir.** Çünkü:
- Az sub_type = LLM'in seçim uzayı dar = yanlış filter olasılığı düşer
- Az sub_type = her sub_type'ın ürün havuzu büyür = filter sonucu kullanılabilir liste döner

### Sub_type tool'a filter olarak ekleniyor mu?

**Evet — iki yerde.**

1. **searchProducts.input.templateSubType** — LLM açıkça koyabilir.
2. **slotExtractor** ([retrieval-service/src/lib/slotExtractor.ts:285-291](retrieval-service/src/lib/slotExtractor.ts#L285-L291)) — kullanıcının query'sinden regex pattern ile inverse map. Örn. "ön yıkama köpüğü" → `prewash_foaming_shampoo`. Bot LLM'i hiç sub_type yazmasa bile microservice query'den çıkarıyor.

slotExtractor'ın bildiği SUB_TYPE_PATTERNS şu an ~30-40 pattern. Sub_type'lar konsolide olunca pattern map de sadeleşir.

### Pratik öneri — "boyut sorgusu" altyapısı için P1 backlog

1. **Volume normalize migration** (Phase 6.5):
   - Tüm ürünler için `specs.volume_ml` (canonical, number) seed et
   - product_meta'ya da projekte et: `(sku, key='volume_ml', value_numeric=...)`
   - capacity_liters/capacity_ml/volume_kg deprecate (canonical migrate edilince UI normalize wizard çalıştır)

2. **Bot tool description güncelle** (Phase 6.5):
   - `metaFilters` describe'ında "Hacim için MUTLAKA `volume_ml` kullan, 1 kg ≈ 1000 ml varsay" ekle
   - Örnek: "25 kg şampuan" → `[{key:'volume_ml', op:'eq', value:25000}]`

3. **slotExtractor volume support** (opsiyonel, ileride):
   - Pattern: `/(\d+)\s*(kg|kilo|litre|lt|ml)/i`
   - Otomatik `volume_ml` slot'u extract et — LLM intent parse'a güvenmeden
   - Birim conversion: kg→ml ×1000, lt→ml ×1000

### Özet

| Kullanıcı sorgusu | Bugün ne olur | Düzeltme sonrası |
|---|---|---|
| "25 kg şampuan" | Vector search'te ad'ında "25" geçen şampuanları getirir, fiyat veya hacim filter'i YOK → unreliable | metaFilters[volume_ml=25000] tam tutarlı 25 kg şampuanlar döner |
| "1 litreden büyük seramik kaplama" | Şu an çalışmaz | volume_ml gte 1000 → kesin |
| "ph nötr şampuan" | Kısmen çalışır (slot extractor "ph nötr" → ph_neutral_shampoo sub) | Hem sub_type hem ph_level metaFilter ile sinerji |

**Kısacası:**
- ✅ Sub_type azaltma SKU bulmayı **kolaylaştırır** (filter precision artar)
- ✅ Meta key standartlaştırma SKU bulmayı **mümkün kılar** (LLM doğru key'i seçer)
- ⚠ Eksik olan tek şey: **specs içindeki volume verisini product_meta'ya da yazma** (migration işi)

---

## 11. Filter strictliği + slot inference riski + carousel seçim mantığı

### A) Sub_type filter ne kadar strict?

**Cevap: Tamamen strict — SQL `AND` koşulu.**

[retrieval-service/src/lib/searchCore.ts:255-260](retrieval-service/src/lib/searchCore.ts#L255-L260) ve [440-448](retrieval-service/src/lib/searchCore.ts#L440-L448):

```sql
WHERE (${tg}::text IS NULL  OR p.template_group = ${tg})
  AND (${tstFamily}::text[] IS NULL OR p.template_sub_type = ANY(${tstFamily}))
  AND (${effectiveBrand}::text IS NULL OR p.brand = ${effectiveBrand})
  ...
```

LLM `templateSubType="paint_coating_kit"` derse → DB'de SADECE paint_coating_kit'ler (1 ürün). paint_coating altındaki 5 ürün gözükmez. **Yanlış sub_type → 0 sonuç riski**.

### B) Mevcut mitigation'lar (4 katman)

| # | Katman | Nerede | Ne yapar |
|---|---|---|---|
| 1 | `expandSubTypeFamily()` | searchCore.ts | Sadece paint_coating ailesi için manuel: paint_coating verilirse 4 alt sub_type genişletilir. **Diğer aileler için patch yok** — ad-hoc fix. |
| 2 | slotExtractor inferred sub_type | slotExtractor.ts:285 | LLM sub_type yazmazsa, query'den regex pattern ile çıkarır. **Explicit LLM > slot** — LLM yazdıysa slot ezilmez. |
| 3 | Bot v10.1 RELEVANCE CHECK | conversations/index.ts:357-364 | LLM yield etmeden önce her ürünün group/sub_type'ı kullanıcı sorgusuyla eşleşiyor mu kontrol eder. **Uyumsuz oran >%30 → tool'u FARKLI parametrelerle yeniden çağırır** (re-tool). |
| 4 | Anti-hallucination | conversations/index.ts:367-379 | Metinde tool output dışı isim yok, yanlış kategoride pekiştirme yok. |

### C) "Sub_type yanlış olsa ürün bulunabilir mi?" — concrete senaryolar

| Senaryo | Bugün ne olur |
|---|---|
| LLM `templateSubType="paint_coating_kit"` (gerçekten kit istemiyor, kullanıcı "seramik kaplama" demişti) | DB'de 1 ürün → carousel boş kalmasın diye LLM v10.1 RELEVANCE CHECK ile re-tool yapar |
| LLM `templateSubType="paint_coating"` (genel) + DB'de Syncro paint_coating_kit altında | `expandSubTypeFamily` patch sayesinde Syncro yine bulunur ✅ |
| LLM hiç sub_type vermedi, slotExtractor "kalın pasta" → heavy_cut_compound | strict filter heavy_cut_compound, %100 doğru sub_type → ✅ |
| slotExtractor pattern HATALI — örn. "şampuan" sözcüğü yanlışlıkla bir sub_type'a map | filter yanlış sub_type → 0 sonuç → LLM v10.1 ile re-tool dener |
| LLM template_group da yanlış (ceramic_coating yerine glass_cleaner_protectant) | Tamamen başka kategori dönerek tool sonucu alır → RELEVANCE CHECK %100 uyumsuz → re-tool ZORUNLU |

→ **Tek katmanlı koruma değil, 4-katman defense in depth.** Ama ilk katmanlar başarısız olursa son katman LLM'in re-tool kararı kalıyor — bu kararın da yanlış olma şansı var (Phase 4 Round 2'de Syncro vakası bu yüzden patch eklendi).

### D) "Sadece template_group kullansak — token maliyeti artar mı?"

**Hayır, AZALIR. Ama precision kaybeder.**

| Faktör | Sub_type ile | Sadece template_group |
|---|---|---|
| Sistem prompt — tool description | ~1100 token (templateGroup enum + templateSubType describe + 157 değer örnekleri) | ~800 token (sadece templateGroup enum) → **−300 token** |
| Tool body (her çağrı) | +~100 token (LLM templateSubType'ı parse + slot doldur) | +0 → **−100 token/çağrı** |
| Recall (kayıp ürün) | Düşük (sub_type strict) | Sıfıra yakın (sadece group filter, vector ranking spesifik tip için) |
| Precision (alakasız ürün) | Yüksek | **Düşük** — "ph nötr şampuan" istesen tüm car_shampoo döner, top-N içine ph nötr olmayanlar girer |
| LLM curator iş yükü | Az (filter çoğu işi yaptı) | **Yüksek** — RELEVANCE CHECK ile çoğu zaman re-tool gerekir |

**Net etki:** Token tasarruf marjinal (~400 token sistem + her çağrıda ~100). Asıl bedel **precision**: aynı sonuç kalitesini almak için LLM 1 çağrı yerine 2-3 çağrı yapmak zorunda kalır → toplam token **artar**.

→ **Sub_type filter'ı kaldırmak değil, KONSOLİDE etmek doğru** (165 → ~110). Spesifiklik korunur, false-negative riski düşer (az opsiyon, az hata).

### E) slotExtractor inference — risky mi?

**Evet, ama mitigated.**

**Risk:** Pattern hatalı match → yanlış sub_type → strict filter → 0 sonuç.

**Mitigation:**

1. **Explicit LLM input slot'u ezer** — LLM templateSubType verdiyse slot devreye girmez ([searchCore.ts:411](retrieval-service/src/lib/searchCore.ts#L411)). Bot doğru parse ederse slot riski yok.

2. **Pattern listesi konservatif** — şu an ~30-40 pattern, her biri spesifik kelime grubu. Generic kelime ("şampuan", "kaplama") ana group'a değil sub_type'a hiç map olmuyor.

3. **Slot phrase strip EDILMIYOR** — query'den çıkarılmıyor ([slotExtractor.ts:289 yorum](retrieval-service/src/lib/slotExtractor.ts#L289)). Yani sub_type filter wrong olsa bile semantic ranking yine query word'leri görür → BM25 hidden context'i kullanır. Bu sadece filter wrong + 0 sonuç senaryosunda işe yaramaz, çünkü filter wrong ise zaten her şey 0.

4. **Bot v10.1 RELEVANCE CHECK** — yine son güvence.

**Pratik test:** Phase 4 Round 2'de slotExtractor "seramik silme bezi" sorgusunda Green Monster'ı (cleaning_cloth) carousel'a soktu → bu BM25/vector match'i (slot değil, content match). Bot RELEVANCE CHECK ile carousel'da gösterse de metinde **"seramik silme bezi olarak Green Monster..." DEMEDİ** (anti-hallucination çalıştı).

### F) Carousel'a konulan ürünler nasıl seçiliyor?

**100% MEKANIK score-based. LLM seçime karışmaz, sadece veto/re-tool yetkisi var.**

#### Pipeline (hybrid mode, [searchCore.ts:362-548](retrieval-service/src/lib/searchCore.ts#L362-L548)):

```
1. Meta filter SQL              (LLM metaFilters → SKU allow-list)
   resolveMetaFilterSkus() · AND semantics across filters
                ↓ (allowSet)
2. Normalize + synonym expand   (turkishNormalize + synonyms tablosu)
                ↓
3. slotExtractor                (brand/priceMin/priceMax/sub_type infer)
                ↓
4. Vector + BM25 PARALEL        (HYBRID_CANDIDATE_LIMIT = ~50 her tarafta)
   ┌─ runBm25Query (Turkish FTS, OR-tsquery)
   └─ pgvector cosine  (filter clause SQL'in içinde)
                ↓
5. RRF fusion (k=60)            (1/(60+rank_bm25) + 1/(60+rank_vec))
                ↓
6. Hydrate top 20-50 SKU        (rerank pool, oversample faktörü)
                ↓
7. applyBusinessBoost           (rrfScore × ratingMult × stockMult × featuredMult)
                ↓ (sıralama BURADA değişebilir RRF'e göre)
8. Top `limit` (default 5)      (ilk 5 alınır)
                ↓
9. exactMatch post-filter       (varsa, ad'da substring match)
                ↓
10. format → carouselItems      (Carousel JSX-ready)
                  productSummaries (raw + similarity skor)
                  textFallbackLines (markdown text)
                ↓
11. JSON response → bot tool sonucu
```

#### Business boost detayı ([searchCore.ts:331-345](retrieval-service/src/lib/searchCore.ts#L331-L345)):

```typescript
finalScore = rrfScore
           × (1 + BOOST_RATING_COEF * rating / 5)
           × (in_stock ? 1.0 : 0.6)        // BOOST_OUT_OF_STOCK
           × (is_featured ? 1.15 : 1.0)    // BOOST_FEATURED
```

Tie break: similarity (cosine), sonra SKU alphabetic.

→ **5 ürünlü carousel = pure deterministic.** Aynı sorguyu 100 kere çalıştırırsan aynı 5 ürün gelir (embed cache de var). LLM **bu seçimi yapmaz**.

#### LLM'in rolü (curator pattern, [conversations/index.ts:355-385](Botpress/detailagent-ms/src/conversations/index.ts#L355-L385)):

LLM tool çağrısından sonra `result.productSummaries` (raw veri + similarity) ve `result.carouselItems` (JSX-ready) alır:

1. **RELEVANCE CHECK** (zorunlu): Her ürünün template_group/sub_type'ı sorguya uyuyor mu?
2. **Karar:**
   - Uyumsuz oran ≤%30 → carousel yield et + metinde uyumsuzları flag'le
   - Uyumsuz oran >%30 → carousel YIELD ETME, tool'u farklı parametrelerle YENİDEN çağır (templateSubType ekle, exactMatch daralt, query reformule)
3. **Anti-hallucination**: Metinde sadece tool output'taki isimleri kullan, kategori uydurma yok.
4. **Variant özetleme**: Tek üründen 3 boyut geldiyse "3 boyut mevcut: 30/50/100ml" diye özetle.

→ LLM **seçici değil, sansürcü**. Skor mekanik, kontrol kalitatif.

---

Bu sayılar tekrar üretilebilir:
```bash
SECRET=$(grep RETRIEVAL_SHARED_SECRET retrieval-service/.env | cut -d= -f2-)
# Sub_type dağılımı
curl -sH "Authorization: Bearer $SECRET" http://localhost:8787/admin/taxonomy \
  | jq '[.groups[].subs[] | .count] | group_by(.) | map({n:.[0], count:length})'
# Duplicate key aileleri
curl -sH "Authorization: Bearer $SECRET" "http://localhost:8787/admin/coverage?limit=80" \
  | jq '.global[] | select(.key | test("capacity|durability|ph|dilution"))'
# FAQ duplicates
curl -sH "Authorization: Bearer $SECRET" "http://localhost:8787/admin/faqs?limit=200" \
  | jq '.items[].question' | sort | uniq -c | sort -rn | head -10
```

---

# §12. Konsolidasyon Yürütme Planı — Multi-Agent Workflow

## Context

§1-11 analizi **ne** yapılması gerektiğini söyledi. §12 **nasıl** ve **kim/hangi ajan** sorularını çözer.

**Kapsam:** 21 template_group (kapsam dışı: marin_products, ppf_tools, fragrance, product_sets, masking_tapes).

**Hedef metrikler:**
| Metrik | Şu an | Hedef |
|---|---|---|
| sub_type sayısı (21 grup) | ~140 | ~95 |
| Distinct meta key | ~520 | ~280 |
| Boş/yanlış sub_type | ~60 (1-2 ürün) | ≤15 |
| FAQ unique question | 600 / 3 156 toplam | aynı + dedup'lı |

**Yöntem:** **Subagent driven** — her faz Claude Code Task ajan'larına paralel/sıralı verilir. İnsan: review + onay + commit. Ajan: analiz + öneri CSV + staging payload.

---

## Faz 0 — Altyapı eksikleri (insan/dev)

| # | Eksik | Konum | Efor |
|---|---|---|---|
| 0.1 | **Group/sub-level toplu key delete** | retrieval-service staging.ts + `/admin/staging/group-key-delete` endpoint | M |
| 0.2 | **Value transform pipeline** (kg→ml, string→number, regex extract) | staging.ts'e `transform` field + executor | M |
| 0.3 | **FAQ CSV batch import** | `/admin/staging/faq-csv` endpoint | S |
| 0.4 | **product_meta projection** (NUMERIC spec keylerini product_meta'ya da yaz) | retrieval-service/src/lib/specsProjection.ts (yeni) + migration | M |

## Faz 1 — Cross-group key normalize (1 ajan)

§8 6 duplicate aile → canonical map → 511 ürün için staging payload CSV.

**Çıktı:** `data/consolidation/phase1-key-normalize.csv` + `phase1-review.csv` (insan onayı).

## Faz 2 — Per-group sub_type konsolidasyon (21 ajan, 6 paralel batch)

| Batch | Gruplar |
|---|---|
| B1 | sprayers_bottles, polishing_pad, microfiber, car_shampoo |
| B2 | spare_part, interior_cleaner, abrasive_polish, ceramic_coating |
| B3 | polisher_machine, storage_accessories, paint_protection_quick, contaminant_solvers |
| B4 | applicators, industrial_products, clay_products, tire_care |
| B5 | leather_care, brushes, glass_cleaner_protectant, glass_cleaner |
| B6 | accessory + orphan |

Her ajan: sub_type merge önerisi + schema base + ürünle ilgisiz key listesi → 3 CSV/grup.

## Faz 3 — Description → specs extract (3 paralel ajan)

511 ürün ÷ 3 ≈ 170 ürün/ajan. Numeric (durability, working_window, ph), Categorical (skill_level, confidence), Boolean (pre_coating_safe) extract → CSV with confidence.

## Faz 4 — Relations madenciliği (1 ajan, Faz 3 sonrası)

Description + FAQ tara → use_with / use_before / use_after / alternatives / accessories önerileri. Bahsedilen ürün adı/marka SKU'ya çöz.

## Faz 5 — FAQ konsolidasyon (1 ajan)

Near-duplicate merge ("Nasıl uygulanır?" + "Nasıl kullanılır?") + §B sinyallerinden yeni FAQ pattern'leri (distribution_channels, solves_problem).

## Faz 6 — Doğrulama

Eval suite + searchByQuery sample + bot RELEVANCE CHECK rate + coverage diff.

---

## Risk + rollback

- Commit'lerden önce DB snapshot (`pg_dump products faqs relations product_meta`)
- Faz 1 ve 2 ajanların CSV çıktısı insan onayı zorunlu
- /commit transaction = all-or-nothing (mevcut)
- Sub_type merge sonrası slotExtractor SUB_TYPE_PATTERNS güncelle

## Yürütme sırası

```
Faz 0 (dev, 2-3 gün) → Faz 1 (1 ajan) → Faz 2 (24 ajan/6 batch)
                                         ↘
                                          Faz 3 + Faz 4 (paralel)
                                                                  ↘
                                                                   Faz 5 → Faz 6
```

---

# §13. Faz 2 REVIZE — Detaylı sub_type merge review + soru-cevap oturumu

## Context — neden tekrar?

**İlk tur ajan yüzeysel karar verdi.** Örnekler:

- `towel_wash → ph_neutral_shampoo` — Kaynak: **mikrofiber bez yıkama şampuanı** (2 ürün, SM MICRON & Q2M TowelWash). Hedef: **araba yıkama şampuanı**. Kullanım senaryosu tamamen farklı (bez vs araba).
- `rinseless_wash → prewash_foaming_shampoo` — Kaynak: **susuz yıkama** (pH nötr, az suyla). Hedef: **agresif ön yıkama köpüğü** (yağ/is/sinek sökücü). Zıt kategori.
- `metal_polish → polish` — Metal yüzey (jant/krom) cilası ≠ boya cilası.
- `tire_coating / wheel_coating → paint_coating` — Lastik/jant boya değil, farklı yüzey.
- `charger → battery` — Şarj aleti batarya değil.
- `felt_pad → wool_pad` — Keçe ped ≠ yün ped, farklı abrasivite.

Ajan, sub_type'daki 2-3 örnek spec'e bakarak yüzeysel semantik benzerlik buldu. **Fonksiyonel uyum + hedef yüzey + kullanım senaryosu** kontrolünü yapmadı.

**Sonuç:** Tüm Faz 2 sub_type merge'leri (~88 değişiklik) 2. tur review'a girer. Faz 2 key-delete (98), tgmerge (8), orphan (6) ayrıca değerlendirilir ama bu revize kapsamında değil.

## Yöntem — 3 paralel ajan + %90 onay rubriği

Kullanıcı talebi: **%90+ emin olmadığın her şeyi kaydet, sonra tek tek sor. Çoğunu sen halle.**

### 5-kriter karar rubriği

Her `source_sub → target_sub` merge için:

| # | Kriter | Onay şartı |
|---|---|---|
| 1 | Primary ürün kategorisi | Kaynak + hedef aynı üst kategori (araba yıkama ≠ bez yıkama) |
| 2 | Hedef yüzey | Aynı yüzey (boya/metal/cam/lastik/plastik — farklı = farklı sub) |
| 3 | Kullanım senaryosu | Aynı iş (ön yıkama ≠ ana yıkama ≠ susuz ≠ bez yıkama) |
| 4 | Formülasyon ailesi | pH + içerik uyumlu (asidik ≠ alkali ≠ nötr; abrasif ≠ non-abrasif) |
| 5 | Filter mantığı | Kullanıcı filter sonucu mantıklı (bot "X ara" → merge sonucu listede makul) |

### Karar kategorileri

- **APPROVE** — 5/5 kriter geçti, %90+ emin → revised staging payload'a ekle
- **REJECT** — 1+ kriter ihlal → iptal + alternatif öner (kalsın / yeni sub / farklı hedef / template_group değişikliği)
- **ASK** — %60-89 emin, belirsiz → kullanıcıya interaktif sor (AskUserQuestion)

### 3 paralel ajan dağılımı

| Ajan | Kapsam | Merge sayısı (yaklaşık) |
|---|---|---|
| **2R-A** | abrasive_polish, applicators, car_shampoo, ceramic_coating, clay_products, contaminant_solvers, interior_cleaner, microfiber | ~46 |
| **2R-B** | paint_protection_quick, polisher_machine, polishing_pad, spare_part, sprayers_bottles, storage_accessories, tire_care | ~15 |
| **2R-C** | leather_care, brushes, glass_care merge (tgmerge), Q2M-PYA4000M taşıma, orphan fix (6 SKU) | ~15 |

## Ajan çıktı formatı (her ajan için)

```
data/consolidation/
├── phase2R-{A,B,C}-approved.csv           (APPROVE, staging-payload)
├── phase2R-{A,B,C}-approved-payload.json  (staging API format)
├── phase2R-{A,B,C}-rejected.md            (REJECT + alternatif)
├── phase2R-{A,B,C}-questions.md           (ASK, soru formatı)
└── phase2R-{A,B,C}-summary.md
```

### Soru formatı (questions.md — kullanıcıya sorulacak):

```
## Q<n>: <grup> / <source_sub> → <target_sub> ?

**Kaynak (n):** SKU1, SKU2 (kısa özet)
**Hedef (n):** SKUa, SKUb (kısa özet)

**Belirsizlik:** <neden %90 değil>

**Alternatifler:**
- A) <source kalsın, benzersiz sub>
- B) <yeni sub aç: X>
- C) <farklı hedef: Y>
- D) <farklı template_group: Z>

**Önerim (zayıf):** <A/B/C/D>
```

## Soru-cevap oturumu

Ajanlar bittikten sonra:

1. 3 ajan'ın `questions.md` dosyalarını birleştir → toplam soru listesi (tahminen 10-30 soru)
2. Kullanıcıya **AskUserQuestion** ile gruplar halinde sor (tool limit ~4 soru/çağrı, birden fazla tur)
3. Her cevaba göre:
   - A → source sub_type kalsın (hiçbir değişiklik yapma)
   - B → yeni sub_type oluştur (staging payload üret: source'tan ürünleri yeni adla güncelle)
   - C → farklı hedefe taşı (staging payload güncelle)
   - D → template_group değişikliği (ek tgmerge payload)
4. Cevapları `phase2R-resolved.csv`'ye kaydet → REVISED payload'a konsolide et

## REVISED payload konsolidasyonu

Soru-cevap tamamlandıktan sonra:

```
data/consolidation/
├── phase2R-FINAL-subtype-merge.csv        (APPROVE + resolved questions)
├── phase2R-FINAL-subtype-merge-payload.json
├── phase2R-FINAL-new-sub_types.md         (yeni oluşturulan sub_type listesi, varsa)
└── phase2R-FINAL-summary.md
```

Bu FINAL payload `/staging/preview` ile doğrulanır. **Onayla sonra commit.**

## Sonraki adım — meta keyler ve detaylar

Kullanıcı: *"Doğru ve yeteri kadar sub_type ayarlarsak sonrasında meta keyleri ve detay bilgileri inceleriz."*

Yani:
1. Faz 2R onaylı + commit → sub_type yapısı doğru
2. Sonra Faz 2 key-delete (98), Faz 1 key-normalize (892), Faz 3 description extract (478), Faz 4 relations (42), Faz 5 FAQ (43) için detaylı review başlat (her biri ayrı aşama)
3. Her aşamada aynı %90 rubriği + soru-cevap pattern'i

## Kritik dosyalar

- Mevcut ilk-tur çıktılar: `data/consolidation/phase2-*-subtype-merge.csv` (input)
- Canlı katalog: `/admin/products` + `/admin/taxonomy` + `/admin/coverage` endpointleri
- Revize çıktılar: `data/consolidation/phase2R-*` (yeni)
- Snapshot (geri alma için): `data/consolidation/_pre-commit-snapshot-20260423-044331/`

## Doğrulama

Ajan çıktıları geldiğinde:

```bash
SECRET=$(grep RETRIEVAL_SHARED_SECRET retrieval-service/.env | cut -d= -f2-)

# Approved payload preview
for f in data/consolidation/phase2R-{A,B,C}-approved-payload.json; do
  curl -sH "Authorization: Bearer $SECRET" -H "Content-Type: application/json" \
    -d @$f http://localhost:8787/admin/staging/preview \
    | jq -c '{file:"'$f'", total, planned, unsupported, skipped}'
done

# Soru sayısı
grep -c "^## Q" data/consolidation/phase2R-{A,B,C}-questions.md
```

## Mevcut durum (bu plan yazılırken)

- 3 ajan arka planda çalışıyor (2R-A, 2R-B, 2R-C)
- Tahmini tamamlanma: 15-20 dakika
- Ajan çıktıları geldikten sonra: soru-cevap oturumu → FINAL payload → commit onay

---

# §14. Phase 2R EKSTRA — `wash_tools` yeni template_group

## Context

Phase 2R FINAL payload'ı (97 change) commit öncesi gözden geçirilirken kullanıcı yeni bir gözlem yaptı: **yıkama ile ilgili ürünler** (eldiven, havlu, sünger, köpük tabancası) farklı gruplarda dağınık duruyor. Bunları tek bir semantik kategoride toplamak LLM için daha temiz filter üretir ve "yıkama aksesuarları" aramalarını düzgün döndürür.

**Mevcut dağılım:**
| Ürün tipi | Şu an | Ürün sayısı |
|---|---|---:|
| Yıkama eldiveni (wash_mitt) | microfiber grubu | 8 |
| Kurulama havlusu (drying_towel) | microfiber grubu | 3 |
| Sentetik güderi (chamois) | microfiber/chamois_drying_towel | 1 (70919) |
| Yıkama süngeri (wash_sponge) | applicators grubu | 1 (SGGD004) |
| Köpük tabancası (foam lance/gun) | sprayers_bottles/foaming_pump_sprayer | 2 (SGGD135, SGGC091) |
| **Toplam** | — | **15** |

## Hedef yapı

**Yeni template_group:** `wash_tools`
**Sub_type'lar (3 sub, 15 ürün):**

1. **`wash_mitt`** (9 ürün) — yıkama eldiveni + süngerleri birleşik:
   - KLIN: 3221A-RD, 3225A-BL, 3227B, 3227I-HD, 3228R
   - GYEON: Q2M-STE, Q2M-WME, Q2M-WP
   - SGCB: SGGD004 (yıkama süngeri — user kararı: mitt'e merge)

2. **`drying_towel`** (4 ürün) — kurulama havluları + güderi:
   - 3213EMR (KLIN Drying DUO EVO)
   - Q2M-SDE7090C (GYEON SilkDryer EVO)
   - Q2M-SODE6080C (GYEON Soft Dryer EVO)
   - 70919 (FRA-BER Sentetik Güderi — chamois drying_towel'a merge)

3. **`foam_lance`** (2 ürün) — köpük tabancası ekipmanları:
   - SGGD135 (SGCB Foam Lance Karcher uyumlu)
   - SGGC091 (SGCB Tornador Foam Gun)

## Phase 2R FINAL ile çakışmalar (düzeltilmeli)

Mevcut `phase2R-FINAL-payload.json` (97 change) içinde iki ilgili değişiklik var:
1. `70919: chamois_drying_towel → drying_towel` (2R-A APPROVED) — **KALIR** ama ek `tg: microfiber → wash_tools` lazım
2. `SGGD004: wash_sponge → cleaning_pad` (2R-A APPROVED) — **İPTAL**; yerine `tg: applicators → wash_tools` + `sub: wash_sponge → wash_mitt`

## Üretilecek ek staging change'ler (17 adet)

| Ürün | Mevcut | Hedef | Change sayısı |
|---|---|---|---:|
| 8 wash_mitt SKU (KLIN + GYEON) | tg=microfiber | tg=wash_tools | 8 tg |
| 70919 chamois | tg=microfiber (2R'de sub zaten drying_towel'a gidiyor) | tg=wash_tools | 1 tg |
| 3 drying_towel SKU | tg=microfiber, sub=drying_towel | tg=wash_tools | 3 tg |
| SGGD004 (wash_sponge) | tg=applicators, sub=wash_sponge | tg=wash_tools, sub=wash_mitt | 1 tg + 1 sub |
| SGGD135 (foam lance) | tg=sprayers_bottles, sub=foaming_pump_sprayer | tg=wash_tools, sub=foam_lance | 1 tg + 1 sub |
| SGGC091 (foam gun) | tg=sprayers_bottles, sub=foaming_pump_sprayer | tg=wash_tools, sub=foam_lance | 1 tg + 1 sub |
| **Phase 2R FINAL'dan İPTAL:** SGGD004 `wash_sponge → cleaning_pad` | — | — | **−1** |
| **Net ekleme** | — | — | **17 change** |

**Yeni FINAL toplam:** 97 + 17 = **114 change** (staging payload)

## Değişecek dosyalar

1. **`data/consolidation/phase2R-FINAL-payload.json`**
   - SGGD004'ün `wash_sponge → cleaning_pad` change'ini SİL
   - 15 ürün için tg değişikliği + 3 ürün için sub değişikliği ekle

2. **`data/consolidation/phase2R-FINAL.csv`** — sync et

3. **`retrieval-service/src/lib/slotExtractor.ts`** — güncelleme:
   - `cleaning_cloth` patternlerinden "koltuk bezi / silme bezi" bölümünü (zaten kapsamıştı, sorun yok)
   - YENİ `wash_mitt` entry: `templateGroup: 'wash_tools'`, patterns: 'yikama eldiveni', 'wash mitt', 'yikama sungeri', 'yikama padi'
   - YENİ `drying_towel` entry (mikrofiber'dan AYIR): `templateGroup: 'wash_tools'`, patterns: 'kurulama bezi', 'kurulama havlusu', 'drying towel', 'waffle havlu', 'guderi', 'güderi', 'chamois'
   - YENİ `foam_lance` entry: `templateGroup: 'wash_tools'`, patterns: 'kopuk tabancasi', 'köpük tabancası', 'foam lance', 'foam gun', 'kopuk yapici', 'köpük yapıcı', 'foam cannon'
   
   Mevcut `microfiber/cleaning_cloth` pattern'i microfiber grubu için kalabilir (Green Monster, interior_cloth, glass_cloth, kit için). Wash_tools tamamen ayrı.

## Türkçe/İngilizce sorusu (kullanıcı sorusu)

Kullanıcı: "user botu türkçe kullanacak fakat kategori ve sub_type isimleri ingilizce, sorun yaratıyor mu?"

**Cevap: HAYIR.** (plan §5'te detaylı — özet:)
- Bot LLM raw sub_type isimlerini görür (İngilizce), ama USER'a GÖSTERMEZ
- User "yıkama eldiveni" yazdığında **slotExtractor** regex pattern ile `wash_mitt` canonical'ına çevirir
- SQL filter sub_type = 'wash_mitt' ile çalışır
- Döndürülen ürün datası formatter tarafından **Türkçe label'larla** carousel'e dönüşür ("Yıkama Eldiveni")
- User hiçbir zaman "wash_mitt" ingilizce string'ini görmez

**Tek koşul:** slotExtractor.ts'de Türkçe regex pattern'ler tam kapsar olmalı (yukarıdaki yeni entry'ler bunu sağlar).

## Doğrulama

```bash
SECRET=$(grep RETRIEVAL_SHARED_SECRET retrieval-service/.env | cut -d= -f2-)
# 1. EK payload'ı preview
jq '{changes: .batches[0].changes}' data/consolidation/phase2R-FINAL-payload.json > /tmp/p2r_flat.json
curl -sH "Authorization: Bearer $SECRET" -H "Content-Type: application/json" \
  -d @/tmp/p2r_flat.json http://localhost:8787/admin/staging/preview | jq '{total, planned, unsupported}'
# Beklenen: total=114, planned=114, unsupported=0

# 2. Commit sonrası smoke test (microservice restart sonrası)
curl -sH "Authorization: Bearer $SECRET" "http://localhost:8787/admin/taxonomy" \
  | jq '.groups[] | select(.group == "wash_tools")'
# Beklenen: wash_tools grubu 3 sub, 15 ürün

# 3. slotExtractor smoke
curl -s "http://localhost:8787/search?q=yikama%20eldiveni" | jq '.items[].templateSubType' | sort -u
# Beklenen: wash_mitt baskın
```

## Uygulama sırası

1. Python script ile phase2R-FINAL-payload.json'a 17 change ekle + SGGD004 mevcut change'i çıkar (yeni toplam 114)
2. Preview ile doğrula (114/114 planned)
3. slotExtractor.ts'ye 3 yeni sub_type pattern entry ekle
4. Kullanıcı final onay
5. Commit + microservice restart + Faz 6 smoke test

## Kritik dosyalar

- `data/consolidation/phase2R-FINAL-payload.json` (modify)
- `data/consolidation/phase2R-FINAL.csv` (sync)
- `retrieval-service/src/lib/slotExtractor.ts` (modify — yeni 3 entry)
- `data/consolidation/phase2R-USER-DECISIONS.md` (append — §14 kararları)

---

## §14b. Meta key normalize — `compatibility` → `target_surface` (KARAR)

User sorusu: "ppf_shampoo'da compatibility, fabric_leather_cleaner'da target_surface, ne alaka?"

**Çözüm:** `compatibility` meta key tamamen kaldırıldı, hepsi `target_surface` (string veya array).

| Önce | Sonra |
|---|---|
| 5 ph_neutral_shampoo: `compatibility=["ppf"]` | `target_surface=["paint","ppf"]` |
| 9 fabric_leather_cleaner: `compatibility=["leather","fabric","plastic"]` | `target_surface=["leather","fabric","plastic"]` |
| Diğer ürünler: `target_surface="leather"` veya `"glass"` (string) | (kalır — string) |

**Bot SQL filter (string OR array contains):**
```sql
WHERE specs->>'target_surface' = 'ppf'
   OR specs->'target_surface' @> '"ppf"'::jsonb
```

Tek meta key, tek mental model.

---

# §15. Phase 2R EKSTRA — spare_part grubunu erit (28 ürün → 3 hedef)

## Context

User backing_plate sorgusundan başlayıp daha kapsamlı bir noktaya getirdi: **spare_part grubu içinde mantıksal olarak farklı 3 küme var.** Bunları ait oldukları ürün gruplarına dağıtmak hem search kalitesi hem cross-sell için iyi.

**Backing_plate yorum güncellemesi:** "Yedek parça değil makine aksesuarı" — eskiyince değişen değil, makineye uygun seçilen ek. Grup taşımayı destekliyor.

**Vector embedding karışıklığı endişesi (cevap):**
- Embedding name + howToUse'a bakar — "Pump Sprayer 1L" ve "Püskürtücü Başlık Yedek" semantik olarak FARKLI vektörler
- BM25 keyword match riski: mitigation `specs.product_type` meta key (container/part/machine/accessory)
- Sub_type doğru belirtildiğinde SQL strict filter → sıfır kirlilik

## 3 küme dağıtımı

### Küme A — polisher_machine'e (13 ürün)

Polisaj makinesine özel aksesuarlar:

| Sub_type | Ürün | Notlar |
|---|---:|---|
| backing_plate | 8 | tabanlık (75/123/125/148/150mm) |
| battery | 1 | FLEX 18V Akü |
| carbon_brush | 2 | FLEX kömür takımı L48/K96 |
| charger | 1 | FLEX 10.8/18V şarj |
| extension_kit | 1 | FLEX uzatma seti |

**Sub_type'lar korunur** (her birinin spesifik anlamı var). Hepsine `specs.product_type=accessory`.

### Küme B — sprayers_bottles'a (13 ürün)

Sprayer/şişe parçaları. User önerisi: handle + trigger_gun aynı sub'da (handle).

| Mevcut sub | Yeni sub | Ürün | Notlar |
|---|---|---:|---|
| trigger_head | trigger_head (korunur) | 4 | EPOCA/FRA-BER püskürtücü başlık |
| trigger_gun | **handle** (merge) | 1 | IK PPF 12 yedek tabanca |
| handle | handle (korunur) | 1 | IK PRO HANDLE |
| hose | hose (korunur) | 2 | IK basınçlı pompa hortum |
| maintenance_kit | maintenance_kit (korunur) | 3 | IK FOAM/MULTI yedek kit |
| nozzle | nozzle (korunur) | 1 | IK PPF nozzle |
| nozzle_kit | nozzle (merge — 2R-A APPROVED zaten) | 1 | IK MULTI nozzle kiti |

User kararı: trigger_gun → handle merge (ikisi de "tutma kolu/tabanca" anlamı).
2R-A APPROVED nozzle_kit → nozzle merge ZATEN payload'da.

Hepsine `specs.product_type=part`.

### Küme C — accessory'ye (2 ürün)

Tornador yedek parçaları (Tornador SGGC055 zaten accessory'ye gidiyor):
- SGYC010 (Tornador Yedek Boncuk)
- SGYC011 (Tornador Yedek Kılcal Hortum)

Sub_type `repair_part` korunur (Tornador-spesifik). `specs.product_type=part`, `specs.compatible_with=tornador`.

## Sonuç

- **spare_part grubu kalkar** (28 → 0 ürün, taxonomy'den otomatik silinir)
- polisher_machine: 23 → 36 ürün (13 yeni accessory)
- sprayers_bottles: 48 → 61 ürün (13 yeni part)
- accessory: 14 → 16 ürün (Tornador yedekleri)

## Ek staging change'ler (~62)

| Tip | Sayı | Detay |
|---|---:|---|
| template_group rename | 28 | 13 spare_part→polisher_machine + 13 spare_part→sprayers_bottles + 2 spare_part→accessory |
| template_sub_type değişiklik | 1 | trigger_gun→handle (1 SKU, nozzle_kit→nozzle zaten payload'da) |
| specs.product_type | 28 | Her ürün için (accessory/part) |
| Mevcut polisher_machine 15 makine için product_type=machine | +15 | Search ayrımı için |
| **Toplam** | **+72** | — |

**Yeni Phase 2R FINAL toplam:** 166 + 72 = **238 change**

## slotExtractor.ts güncelleme

Yeni pattern'ler:
- `backing_plate` (templateGroup=polisher_machine): "tabanlık", "backing plate", "pad destek", "velcro taban"
- `trigger_head` (templateGroup=sprayers_bottles): "yedek başlık", "püskürtücü başlık", "trigger head", "spray head"
- `nozzle` (templateGroup=sprayers_bottles): "yedek nozzle", "nozzle"
- `maintenance_kit` (templateGroup=sprayers_bottles): "bakım kiti", "yedek kit", "maintenance kit"
- `hose` (templateGroup=sprayers_bottles): "hortum", "uzatma hortumu"
- `handle` (templateGroup=sprayers_bottles): "tabanca", "kol", "yedek tabanca"
- `battery` (templateGroup=polisher_machine): "yedek akü", "akü", "battery"
- `charger` (templateGroup=polisher_machine): "şarj cihazı", "charger"
- `carbon_brush` (templateGroup=polisher_machine): "kömür takımı", "yedek kömür"

## Doğrulama

```bash
SECRET=$(grep RETRIEVAL_SHARED_SECRET retrieval-service/.env | cut -d= -f2-)
jq '{changes: .batches[0].changes}' data/consolidation/phase2R-FINAL-payload.json > /tmp/p2r.json
curl -sH "Authorization: Bearer $SECRET" -H "Content-Type: application/json" \
  -d @/tmp/p2r.json http://localhost:8787/admin/staging/preview | jq '{total, planned, unsupported}'
# Beklenen: total=238, planned=238, unsupported=0
```

## Bot kullanım senaryoları

| User sorgu | Beklenen davranış |
|---|---|
| "polisaj makinesi öner" | bot: `templateGroup=polisher_machine + metaFilter[product_type=machine]` → 15 makine, accessory'ler hariç |
| "rotary polisaj için tabanlık" | slotExtractor: tg=polisher_machine + sub=backing_plate → 8 tabanlık |
| "sprayer yedek başlığı" | slotExtractor: tg=sprayers_bottles + sub=trigger_head → 4 başlık |
| "PPF tabancam için nozzle" | slotExtractor: tg=sprayers_bottles + sub=nozzle → IK nozzle ürünleri |
| "FLEX şarj cihazı" | slotExtractor: tg=polisher_machine + sub=charger → 1 ürün |
| "Tornador yedek boncuk" | slotExtractor: tg=accessory + sub=repair_part → SGYC010 |

## Değişecek dosyalar

- `data/consolidation/phase2R-FINAL-payload.json` — 72 ek change
- `data/consolidation/phase2R-FINAL.csv` — sync
- `retrieval-service/src/lib/slotExtractor.ts` — 9 yeni pattern entry

**Mevcut 8 backing_plate ürün:**
- 26931.099.001 — MENZERNA Polisaj Tabanlığı (Kırmızı) 123mm
- 26934.099.001 — MENZERNA Rotary için Tabanlık 75mm
- 26935.099.001 — MENZERNA Rotary için Esnek Tabanlık 148mm
- 487988 — FLEX XCE/XFE Orbital Velcro Taban 150mm
- 492361 — FLEX BP-M D75 PXE 80 için Cırtlı Tabanlık 75mm
- SGGD053 — SGCB Rotary Esnek Silikon Pad Tabanı 150mm
- SGGD166 — SGCB Mini Polisaj Tabanlık Seti 2 Adet
- SGGF181-57 — SGCB Orbital Polisaj Tabanı 125mm

## Hedef yapı

**Taşıma:** template_group `spare_part` → `polisher_machine`, sub_type `backing_plate` korunur.

**Search kirliliği mitigation (zorunlu):**
- Tüm 23 polisher_machine ürünü için `specs.product_type` meta key ekle:
  - `"machine"` (15 makine: rotary, orbital, dual_action_polisher, sander, machine_kit)
  - `"accessory"` (8 backing_plate)
- Bot `metaFilters: [{key:"product_type", op:"eq", value:"machine"}]` ile makine arar, backing_plate karışmaz.

**slotExtractor.ts pattern:**
- Yeni `backing_plate` entry, templateGroup=polisher_machine
- Patterns: 'tabanlik', 'tabanlık', 'backing plate', 'pad destek', 'velcro taban', 'pad support', 'polisaj tabani', 'polisaj tabanı'

## Ek staging change'ler (+24)

| Ürün | Değişiklik | Sayı |
|---|---|---:|
| 8 backing_plate SKU | tg `spare_part` → `polisher_machine` | 8 |
| 8 backing_plate SKU | specs.product_type=accessory | 8 |
| 15 polisher_machine makine | specs.product_type=machine | 8 |

Polisher_machine 15 makine için product_type=machine: rotary (4), orbital (4), dual_action_polisher (4), sander (2), machine_kit (1) = 15. Toplam +24 change.

**Yeni Phase 2R FINAL:** 166 + 24 = **190 change**.

## Değişecek dosyalar

1. `data/consolidation/phase2R-FINAL-payload.json` — 24 ek change
2. `data/consolidation/phase2R-FINAL.csv` — sync
3. `retrieval-service/src/lib/slotExtractor.ts` — yeni `backing_plate` pattern entry

## Doğrulama

```bash
SECRET=$(grep RETRIEVAL_SHARED_SECRET retrieval-service/.env | cut -d= -f2-)
jq '{changes: .batches[0].changes}' data/consolidation/phase2R-FINAL-payload.json > /tmp/p2r.json
curl -sH "Authorization: Bearer $SECRET" -H "Content-Type: application/json" \
  -d @/tmp/p2r.json http://localhost:8787/admin/staging/preview | jq '{total, planned, unsupported}'
# Beklenen: total=190, planned=190, unsupported=0
```

## Bot kullanım senaryoları

| User sorgu | Beklenen davranış |
|---|---|
| "polisaj makinesi öner" | bot: `templateGroup=polisher_machine + metaFilter[product_type=machine]` → 15 makine |
| "rotary polisaj makinesi" | slotExtractor: templateSubType=rotary → 4 ürün |
| "FLEX için tabanlık" | slotExtractor: templateSubType=backing_plate → 8 tabanlık (FLEX exact match en üste) |
| "75mm tabanlık" | metaFilter[diameter_mm=75] + sub=backing_plate → 26934 + 492361 |

---

# §16. Commit sonrası yol haritası — YÜRÜTME SIRASI

## Context

Phase 2R FINAL payload **325/325 planned** durumda. Bütün sub_type review turu tamamlandı (15 grup detaylı incelendi, 14 AskUserQuestion turu, §14b compatibility→target_surface, §15 spare_part eritildi, masking_tapes trim_tape merge). Sıra **commit + doğrulama + bot instruction güncellemesi**.

## Adım adım yürütme (8 adım)

### Adım 1 — Pre-commit preview doğrulaması (READ-ONLY)
```bash
SECRET=$(grep RETRIEVAL_SHARED_SECRET retrieval-service/.env | cut -d= -f2-)
jq '{changes: .batches[0].changes}' data/consolidation/phase2R-FINAL-payload.json > /tmp/p2r_final.json
curl -sH "Authorization: Bearer $SECRET" -H "Content-Type: application/json" \
  -d @/tmp/p2r_final.json http://localhost:8787/admin/staging/preview \
  | jq '{total, planned, unsupported, skipped}'
```
**Beklenen:** `{total:325, planned:325, unsupported:0, skipped:0}`. Sapma varsa durup analiz.

### Adım 2 — 3 katmanlı yedek (rollback güvencesi)

Commit **Supabase Postgres'e doğrudan yazar** (retrieval-service SUPABASE_DB_URL ile bağlı). `/admin/staging/commit` endpoint'i atomic transaction — ya hep ya hiç. Ama yine de **üç katman yedek alınır**:

**(a) Supabase Dashboard otomatik yedek** (zaten var)
Supabase dashboard → Database → Backups → son 7 gün Point-in-Time Recovery.

**(b) pg_dump manuel (önerilen — "tam kurtarma" için)**
```bash
# Supabase connection string'i .env'den al
DB_URL=$(grep SUPABASE_DB_URL retrieval-service/.env | cut -d= -f2-)
TS=$(date +%Y%m%d-%H%M%S)
mkdir -p data/consolidation/_pg-dump-$TS

# Sadece bu 5 tablo (taxonomy ile ilgili, hepsi cascade)
pg_dump "$DB_URL" \
  --data-only --inserts \
  -t products -t product_meta -t product_faqs -t product_relations -t product_search \
  -t product_embeddings \
  > data/consolidation/_pg-dump-$TS/taxonomy-tables.sql
```
Rollback gerekirse: `psql "$DB_URL" < taxonomy-tables.sql` (önce TRUNCATE gerekir).

**(c) API snapshot (JSON, reverse-staging için)**
```bash
TS=$(date +%Y%m%d-%H%M%S)
mkdir -p data/consolidation/_pre-commit-phase2R-$TS
curl -sH "Authorization: Bearer $SECRET" "http://localhost:8787/admin/products?limit=2000" \
  > data/consolidation/_pre-commit-phase2R-$TS/products.json
curl -sH "Authorization: Bearer $SECRET" "http://localhost:8787/admin/taxonomy" \
  > data/consolidation/_pre-commit-phase2R-$TS/taxonomy.json
```
Bu JSON'lardan **reverse-staging payload** üretilebilir (before/after swap → 325 change'i geri al).

**Öneri:** En az (b) pg_dump + (c) API snapshot. (a) Supabase otomatik yedek genel emniyettir.

### Adım 3 — COMMIT (tek transaction, 325 change)
```bash
curl -sH "Authorization: Bearer $SECRET" -H "Content-Type: application/json" \
  -d @data/consolidation/phase2R-FINAL-payload.json \
  http://localhost:8787/admin/staging/commit | jq
```
**Beklenen:** `{ok:true, applied:325, errors:0}`. Transaction atomic — tek bir satır hata verse tüm batch rollback olur.

### Adım 4 — JC0101 manual DELETE (Supabase SQL Editor, kullanıcı)
`data/consolidation/phase2R-EXTRAS-sql.md` §1'deki DELETE cascade scripti Supabase dashboard'dan elle çalıştır. Staging bu işlemi desteklemiyor (scope='product' DELETE yok).

### Adım 5 — KRİTİK: search_text yenile + re-embed + slotExtractor güncelle

**Bağlam — neden kritik?** Staging commit sadece `products` ve `product_meta` tablolarını günceller. 3 türev veri katmanı **otomatik yenilenmez**:

| Katman | Ne saklar | Stale sonucu |
|---|---|---|
| `product_search.search_text` | denormalize metin (name + brand + sub_type + howToUse + ...) | BM25 FTS eski sub_type kelimesini arar, yeni kategoride bulamaz |
| `product_search.search_vector` | Turkish tsvector (search_text'ten generated) | Otomatik yenilenir ÇÜNKÜ `GENERATED ALWAYS AS` — ama search_text güncellenmedikçe faydasız |
| `product_embeddings.embedding` | Gemini 768d vector (search_text'ten embed) | Vector search eski sub_type semantiğini hatırlar — "yıkama eldiveni" aramasında microfiber cluster'ına yakın kalır |

**325 change'in ~280'i** template_group veya template_sub_type'ı değiştiriyor → search_text + embedding **hepsi için yenilenmeli**. Basit yol: **tüm 511 ürün için yeniden üret** (~2 dakika).

**5a — search_text regenerate (bulk upsert):**
```bash
cd retrieval-service

# search_text şablonu = name + brand + sub_type + howToUse (seed-products.ts'teki formül)
# Basit yol: seed-products.ts'nin search_text bölümünü yeniden çalıştırmak,
# ama mevcut script CSV'den okuyor. Live DB'den regenerate için ayrı script gerekir.

bun run scripts/regenerate-search-text.ts  # (YAZILACAK — alt bölüme bak)
```

**Önerilen regenerate script** (~30 satır, `retrieval-service/scripts/regenerate-search-text.ts`):
```ts
// Live products tablosundan search_text üret, product_search'e UPSERT
import { sql } from '../src/lib/db.ts';

const rows = await sql`
  SELECT sku, name, brand, template_group, template_sub_type, 
         short_description, how_to_use, when_to_use
  FROM products
`;
for (const r of rows) {
  const searchText = [
    r.name, r.brand, r.template_group, r.template_sub_type,
    r.short_description, r.how_to_use, r.when_to_use,
  ].filter(Boolean).join(' | ');
  
  await sql`
    INSERT INTO product_search (sku, search_text) VALUES (${r.sku}, ${searchText})
    ON CONFLICT (sku) DO UPDATE SET search_text = EXCLUDED.search_text
  `;
}
```

**5b — Re-embed tüm ürünler:**
```bash
# En basit yol: EMBEDDING_VERSION bump → script zaten resumable
# retrieval-service/src/lib/embed.ts → EMBEDDING_VERSION = 'gemini-embedding-001-v1' → 'v2'

# Veya product_embeddings'i boşalt (daha surgical):
DB_URL=$(grep SUPABASE_DB_URL retrieval-service/.env | cut -d= -f2-)
psql "$DB_URL" -c "TRUNCATE product_embeddings"

cd retrieval-service && bun run scripts/embed-products.ts
# Beklenen çıktı: "511 products need embedding" → ~2 dakika
```

**5c — slotExtractor.ts güncellemesi**
`retrieval-service/src/lib/slotExtractor.ts` dosyasına **~25 yeni pattern entry** ekle (§14, §15'te listelenen: wash_mitt, drying_towel, foam_tool, backing_plate, trigger_head, nozzle, maintenance_kit, hose, handle, battery, charger, carbon_brush, rotary, orbital, dual_action_polisher, sander, cleaning_cloth, fabric_leather_cleaner, leather_dressing, surface_prep, tire_dressing, applicator_pad, heavy_cut_compound, polish, trim_tape).

**5d — Microservice restart**
```bash
pkill -f "bun run.*server.ts" && cd retrieval-service && bun run src/server.ts &
```

### Adım 6 — Post-commit doğrulama (READ-ONLY)
```bash
# 6a — Taxonomy yeni yapı
curl -sH "Authorization: Bearer $SECRET" "http://localhost:8787/admin/taxonomy" \
  | jq '.groups | map({g:.group, subs:(.subs | length), total:(.subs | map(.count) | add)})'
# Beklenen: wash_tools (3 sub, 15 ürün), spare_part YOK, polisher_machine (36 ürün), vb.

# 6b — slotExtractor smoke
curl -s "http://localhost:8787/search?q=yikama%20eldiveni" | jq '.items[].templateSubType' | sort -u
# Beklenen: wash_mitt baskın

curl -s "http://localhost:8787/search?q=tabanlik" | jq '.items[].templateSubType' | sort -u
# Beklenen: backing_plate

curl -s "http://localhost:8787/search?q=lastik%20parlatici" | jq '.items[].templateSubType' | sort -u
# Beklenen: tire_dressing
```

### Adım 7 — Admin UI görsel doğrulama (kullanıcı)
Next.js admin-ui'yi aç, kontrol et:
- **Taxonomy tree** — spare_part yok, wash_tools var, polisher_machine 36 ürün
- **Product detail sayfası** — örn. SGGD402 (eski water_spray_gun, şu an kept), 70919 (eski chamois, şu an wash_tools/drying_towel), 8 backing_plate SKU (şimdi polisher_machine)
- **Specs tab** — target_surface doğru (array/string), product_type (machine/accessory/part) doğru

### Adım 8 — Bot instruction/prompt güncellemesi
**Değişen dosyalar:**
- `Botpress/detailagent-ms/src/conversations/index.ts` (~541 satır, v10.1 instructions)
- `Botpress/detailagent-ms/src/tools/*.ts` (searchProducts, searchByRating, vb. tool descriptions)

**Güncellenecek noktalar (özet):**
| Konu | Mevcut | Yeni |
|---|---|---|
| template_group enum sayısı | 26 (spare_part dahil) | 25 (spare_part yok, wash_tools eklendi) |
| `templateGroup='product_sets'` uyarısı (line 187) | — | Hâlâ geçerli (değişmedi) |
| `paint_coating_kit → paint_coating` aile expansion (line 362) | expandSubTypeFamily patch'i ile kompanse ediliyor | Artık direkt paint_coating (kit/multi_step/single_layer eridi) |
| `heavy_cut_compound` — gyeon_glass_polish, ppf_renew | heavy_cut_compound altında | **polish** sub_type'ına taşındı |
| `fabric_leather_cleaner` → meta key | compatibility=[...] | **target_surface=[...]** |
| `ppf_shampoo` | compatibility=["ppf"] | **target_surface=["paint","ppf"]** |
| `trim_tape` | detailing_tape vs trim_tape ikilem | Tek `trim_tape` |
| `spare_part` → polisher_machine/sprayers_bottles/accessory ayrımı | — | metaFilter `product_type=machine\|accessory\|part` ile ayrım ekle |
| `searchProducts metaFilters describe` | generic örnekler | Canonical key listesi (target_surface, product_type, purpose, formulation, has_ceramic, is_kit, pack_size) + örnekler |

**Yeni bot davranışları tanımla:**
- User "polisaj makinesi öner" → `templateGroup=polisher_machine + metaFilter[product_type=machine]` (accessory karışmasın)
- User "yıkama eldiveni" → `templateGroup=wash_tools + templateSubType=wash_mitt`
- User "PPF şampuanı" → `templateGroup=car_shampoo + metaFilter[target_surface contains ppf]`

## Doğrulama kriterleri (done tanımı)

- [ ] Commit response `applied:325, errors:0`
- [ ] Post-commit taxonomy: 25 group (spare_part yok), wash_tools var
- [ ] slotExtractor smoke: wash_mitt, backing_plate, tire_dressing sorguları doğru sub_type dönüyor
- [ ] Admin UI: Ürün detayları yeni tg/sub ile görünüyor, specs.target_surface/product_type tutarlı
- [ ] JC0101 DB'den silindi (4 tabloda 0 satır)
- [ ] Bot instructions: yeni taxonomy'ye göre güncellendi, eval seti bir kez çalıştırıldı

## Sonraki fazlar (bu iş bitince, ayrı commit'ler)

| Faz | Kapsam | Change sayısı |
|---|---|---:|
| **Faz 3R** | Description → specs extract review (ph_level, durability_months, working_window, skill_level) | ~84 |
| **Faz 4** | Relations (use_with, use_before, alternatives, accessories) | ~42 |
| **Faz 5** | FAQ near-duplicate merge ("Nasıl uygulanır?" + "Nasıl kullanılır?") | ~43 |
| **Faz 6** | Smoke test + eval suite full run | — |

Bu 3 faz Phase 2R commit'i başarılı olunca açılır. Her biri ayrı staging payload + ayrı review + ayrı commit.

## Risk + rollback

**3 rollback senaryosu:**

| Durum | Geri dönüş yolu | Süre |
|---|---|---|
| Commit kısmen başarısız | Atomic transaction — kendiliğinden rollback, hiçbir değişiklik uygulanmaz | anında |
| Commit başarılı, ama taxonomy yanlış çıktı | **pg_dump restore**: `psql "$DB_URL" < data/consolidation/_pg-dump-*/taxonomy-tables.sql` (önce TRUNCATE) | ~2 dakika |
| Commit + embedding OK ama bot davranışı bozuk | Reverse-staging payload üret (API snapshot'tan) → /admin/staging/commit | ~5 dakika |
| Her şey tamamen bozuk | **Supabase Dashboard → Backups → Point-in-Time Restore** (önceki gün) | ~15 dakika, tüm DB |

**Embedding rollback:** Adım 5b'de EMBEDDING_VERSION bump yaparsan eski embedding'ler `product_embeddings`'de kalır (upsert, silinmez). Geri dönmek için EMBEDDING_VERSION'ı eski değere çevir.

**Search_text rollback:** Eski seed CSV (`data/seed/product_search_index.csv`) varsa oradan restore edilir, yoksa pg_dump'tan.

## Kritik dosyalar

| Dosya | Rol |
|---|---|
| `data/consolidation/phase2R-FINAL-payload.json` | 325 change, staging API payload |
| `data/consolidation/phase2R-FINAL.csv` | İnsan okunabilir sync |
| `data/consolidation/phase2R-EXTRAS-sql.md` | JC0101 DELETE + re-embed |
| `retrieval-service/src/lib/slotExtractor.ts` | ~25 yeni pattern entry |
| `Botpress/detailagent-ms/src/conversations/index.ts` | Bot v10.1 instructions |
| `Botpress/detailagent-ms/src/tools/search-products.ts` | Tool descriptions |

---

# §17. Meta key canonical migration — DOĞRULANMIŞ plan (Faz 1)

## Context

Kullanıcı planı revize etti. Plan §8'in yüzeysel önerileri DB'den live data ile karşılaştırıldı; bazı öneriler yanlış çıktı. Kullanıcı kararları + gerçek ürün sayıları + iş bilgisi ile aşağıdaki canonical map kesinleşti.

**Kullanıcı kararları (özet):**
1. **fragrance + product_sets:** Dokunulmasın
2. **specs.features:** Kalsın (genel kullanım, 169 üründe var)
3. **specs.sub_type:** Sil (template_sub_type kolonu ile çakışma)
4. **durability_washes:** Sil — ama 1 dolu ürün var (Q2-TYA500M=5+ Yıkama), karar gerek
5. **dilution nested:** Object yapı kullan, eksik değerler **boş kalsın** (uydurma yok)
6. **coverage_per_sqm_ml:** Sil (1 ürün, gereksiz). **Seramik kaplamalar için global kural:** 25ml/otomobil, 15ml/motosiklet — bu bot instruction'a girer, per-product key gereksiz

## Aile A — Volume / Capacity

**İki ayrı kavram:**
- **volume** = ürünün içeriği (ml/kg satılan miktar)
- **capacity** = sprayer/şişe tankı (kaç ml dolar)

| Canonical | Tip | Hangi ürünlerde | Migration |
|---|---|---|---|
| `volume_ml` | number | Tüm sıvı/jel ürünler (511 ürünün ~480'i) | volume_liters × 1000, volume_kg × 1000 (1 kg ≈ 1000 ml yaklaşımı) |
| `capacity_ml` | number | Sprayer/şişe (sadece sprayers_bottles grubu, ~48 ürün) | capacity_liters × 1000, capacity_total_lt × 1000 |
| `capacity_usable_ml` | number (opsiyonel) | Pump sprayer'lar için güvenli doldurma sınırı | capacity_usable_lt × 1000 |

**Silinecek aliases:** `volume_liters`, `volume_kg`, `capacity_liters`, `capacity_total_lt`, `capacity_usable_lt`

## Aile B — Durability

| Canonical | Tip | Migration kuralı |
|---|---|---|
| `durability_months` | number | durability_weeks ÷ 4.33; durability_days ÷ 30; durability_label parse ("2 yıl"→24, "uzun süreli"→null) |
| `durability_km` | number (opsiyonel) | Mevcut korunur, sadece km bazlı söylenenler |

**Silinecek:** `durability_label` (numerik'e çevrildikten sonra), `durability_weeks`, `durability_days`, `durability_washes`

**⚠ ÇÖZÜM GEREKEN — durability_washes (1 ürün dolu):**
- `Q2-TYA500M` (GYEON Q Tire) → `durability_washes=5+ Yıkama`
- **Sor:** Bu bilgi nereye?
  - (a) `durability_months ≈ 1` (5 yıkama ≈ haftada 1 = ~1 ay)
  - (b) `durability_label="5+ yıkama"` korunsun (ama label'ı silmeyi de söyledin)
  - (c) At, tire_dressing semantik info zaten yeterli

## Aile C — pH

| Canonical | Tip | Anlamı |
|---|---|---|
| `ph_level` | number (1-14) | Ürünün **kendi kimyasal pH'ı** (asidik/nötr/alkali) |
| `ph_tolerance` | string range "2-11" | Kaplama ürünlerinin **dayanabileceği yüzey pH aralığı** (üzerine asidik şampuan vs.) |

**Silinecek aliases:** `ph` (eski isim), `ph_label` (ph_level'dan türetilebilir: <6=asidik, 6-8=nötr, >8=alkali)

## Aile D — Dilution (nested object)

**Kullanıcı onayı:** Nested object kullan. Veri olmayan değerler **boş bırakılsın, uydurma yok**.

| Canonical | Tip | Anlamı |
|---|---|---|
| `dilution` | nested object | Uygulama metoduna göre seyreltme oranları |

```jsonc
dilution: {
  bucket: "1:200",        // Kova ile yıkama (varsa)
  foam_lance: "1:50",     // Köpük tabancası (varsa)
  pump_sprayer: "1:30",   // Pump sprayer (varsa)
  manual: "1:20"          // Sünger/bez (varsa)
}
```

**Kural:** Sadece üretici/etiket bilgisi varsa doldur. Yoksa key yok (null veya yok yazma — eksik bırak). Bot instruction'da "dilution.bucket yoksa 'üretici belirtmemiş' de" denir.

**Silinecek aliases:** `dilution`, `dilution_ratio`, `dilution_scale`, `dilution_steps_ml`, `dilution_kova` (TR→EN), `recommended_bucket_ml` (zaten 8 üründe BOŞ — sil)

**Migration:** 
- `dilution_kova="1:200"` → `dilution.bucket="1:200"`
- `dilution_foam_lance="1:50"` → `dilution.foam_lance="1:50"`
- `dilution_pump`, `dilution_manual` → karşılık gelen alt key

## Aile E — Consumption (REVİZE — kullanıcı son kararları)

**Kullanıcı düzeltmesi:** consumption_per_car_ml SADECE şampuan değil, **araç başı ölçülebilir tüketimi olan TÜM ürünler** için yazılır. Per-product değer kaydedilir.

**⚠ DÜZELTME (motorcycle key UYDURMASI iptal):** Önceki versiyonda `consumption_per_motorcycle_ml` per-product key olarak öneriliyordu — bu DB'de YOK ve kullanıcı onaylamadı. Doğru karar: motosiklet kuralı **bot instruction'a** girer (15 ml/moto global), per-product key DEĞİL.

| Canonical | Tip | Hangi ürünlerde | Default değer |
|---|---|---|---|
| `consumption_per_car_ml` | number | Şampuanlar (5 sub) + ceramic_coating + tire_dressing + fabric_coating + bazı diğerleri | her ürün için spesifik |

**Default değerler (üretici/ölçüm bilgisi yoksa kullanılır):**
| Ürün ailesi | consumption_per_car_ml |
|---|---|
| `ceramic_coating` (paint_coating, vb.) | 25 (otomobil ortalama) |
| `tire_dressing` | 10 (Gyeon Tire Express, Gyeon Tire) |
| `ph_neutral_shampoo` | 17-25 (üreticiye göre) |
| `prewash_foaming_shampoo` | 50 (köpük formulasyonu daha yoğun) |
| `fabric_coating` (Q2-FCNA400M) | 150 (kumaş kaplama daha bol) |

**Motosiklet için (bot instruction):** Seramik kaplamalar için motosiklet tüketimi = 15 ml. Bot bu kuralı kendi bilir, per-product DB key'i YOK.

**Migration (üretici verisi varsa onu kullan, yoksa default):**
- 54 mevcut `consumption_ml_per_car` → `consumption_per_car_ml` rename + boş kalanlar default
- ceramic_coating ~23 ürün için → 25 default seed (özel ölçüm yoksa)
- tire_dressing ~7 ürün için → 10 default seed
- Q2-FCNA400M → 150 (kullanıcı kararı)

**Silinecek (kullanıcı onayı):**
- `consumption_ml_per_cabin` (1 ürün Q2-LSE50M, kullanıcı: "bu üründe olmasın saçma olur") → SİL
- `coverage_ml_per_sqm` (1 ürün Q2-FCNA400M=80) → consumption_per_car_ml=150'ye dönüş
- `recommended_foam_cannon_ratio` (8 ürün hepsi BOŞ) → SİL
- `recommended_bucket_ml` (8 ürün hepsi BOŞ) → SİL

**Bot için açıklama (instruction'a girecek):**
> `consumption_per_car_ml` her ürün için DB'de yazılı. Bot LLM bu key'i metaFilter veya hesaplama için kullanır. Şişe başı araç sayısı = `volume_ml ÷ consumption_per_car_ml`. Örn: Q2-PCEVO50M (volume_ml=50, consumption=25) → 2 araç.
>
> **Motosiklet için (sadece seramik kaplama):** Motosiklet sayısı = `volume_ml ÷ 15` (per-product key YOK, global kural). Örn: 50 ml seramik → 3 motosiklet.

## Aile F — Substrate / Compatibility / Target Surface (3 ayrı kavram)

Plan §14b'de zaten `compatibility → target_surface` migration başlatıldı (5 ppf_shampoo + 9 fabric_leather_cleaner). Ama §8'in "hepsini compatibility'ye topla" önerisi yanlış. Üç kavram NET ayrılmalı:

| Canonical | Tip | Anlamı | Örnek |
|---|---|---|---|
| `target_surface` | string \| string[] | Ürünün **ASIL kullanıldığı yüzey** | leather_cleaner: "leather"; ppf_shampoo: ["paint","ppf"] |
| `compatibility` | string[] | Ürünün **üzerine uygulanabileceği mevcut katman** | top_coat: ["ceramic_coating"]; quick_detailer: ["ceramic_coating","ppf"] |
| `substrate_safe` | string[] | Ürünün **zarar vermediği alt malzeme** | wheel_cleaner: ["aluminum","fiberglass","plexiglass"] |

**Migration:**
- `safe_on_ceramic_coatings: true` → `compatibility: [...,"ceramic_coating"]`
- `safe_on_ppf_wrap: true` → `compatibility: [...,"ppf"]`
- `aluminum_safe: true` → `substrate_safe: [...,"aluminum"]`
- `fiberglass_safe: true` → `substrate_safe: [...,"fiberglass"]`
- `plexiglass_safe: true` → `substrate_safe: [...,"plexiglass"]`

## Aile G — products.specs.sub_type (silme)

`specs.sub_type` JSONB içinde `template_sub_type` kolonunun **kopyası**. 511 üründe var, kullanılmıyor.

**Migration:** Tüm 511 ürün için `specs` JSONB'sinden `sub_type` field'ını sil (`specs - 'sub_type'`).

## Yürütme — kaç change?

| Aile | Etkilenen ürün | Tahmini change |
|---|---|---|
| A. Volume/capacity | ~480 | ~480 (her ürün 1 key rename + alias delete) |
| B. Durability | ~50 | ~50 |
| C. pH | ~80 | ~80 |
| D. Dilution | ~30 | ~30 |
| E. Consumption | ~70 | ~70 (54 rename + 16 alias delete) |
| F. Substrate/compat | ~25 | ~25 |
| G. specs.sub_type drop | 511 | 511 |
| **Toplam Faz 1** | — | **~1250 change** |

Bu Faz 2R (325 change) + Faz 1 (~1250 change) = **~1575 change**, tek payload, tek commit, tek re-embed.

## Doğrulama (commit ÖNCESİ)

```bash
# Her aile için "öncesi-sonrası" sample SKU sorgusu
SECRET=$(grep RETRIEVAL_SHARED_SECRET retrieval-service/.env | cut -d= -f2-)
for sku in 26005 Q2-TYA500M Q2M-PPFW500M; do
  curl -sH "Authorization: Bearer $SECRET" "http://localhost:8787/admin/products/$sku" \
    | jq '{sku, name, specs: (.specs | with_entries(select(.key | test("volume|capacity|ph|dilution|consumption|target_surface|compatibility|substrate"))))}'
done
```

## ⚠ Karar tablosu (kullanıcı cevapladı — son durum)

| # | Soru | Karar |
|---|---|---|
| 1 | Q2-TYA500M `durability_washes=5+ Yıkama` | **`durability_km=1000`** olarak yeniden yaz (Gyeon Tire için), washes key sil |
| 2 | Q2-LSE50M `consumption_ml_per_cabin=35` (leather_coating) | **At, hiçbir consumption key yazma** ("saçma olur") |
| 3 | Q2-FCNA400M `coverage_ml_per_sqm=80` (fabric_coating) | **`consumption_per_car_ml=150`**, sqm key sil |
| 4 | kg → ml çevirisi | **1:1** (karşılaştırma için, density tablosu gereksiz) |
| 5 | specs.features yapısı | **Serbest metin (string) kalsın** — mevcut hâl korunur, migration yok |

## Sonraki sıra (commit öncesi)

1. ✅ 5 sorunun cevabı alındı (yukarıdaki tablo)
2. **Faz 1 ajan koşusu** — 7 aile için detaylı staging payload (~1250 change)
   - Her aile için ayrı ajan, kullanıcı kararlarına sıkı bağlı
   - Çıktı: `data/consolidation/phase1-{A..G}-payload.json` + `phase1-{A..G}-summary.md`
3. **Faz 1 Review** — ajan çıktıları, sample SKU'ları kullanıcıya göster, onay al
4. **Faz 3 ajan koşusu** — description → specs extract (~84 change, 3 paralel ajan)
5. **Faz 4 + Faz 5 paralel** — relations (~42) + FAQ dedup (~43)
6. **Tüm payload'ları birleştir:**
   - Faz 2R FINAL (325) + Faz 1 (~1250) + Faz 3 (~84) + Faz 4 (~42) + Faz 5 (~43) = **~1744 change**
   - Birleştirme scripti: `data/consolidation/build-mega-payload.ts`
   - Conflict resolution: aynı SKU+field için son ID kazanır (öncelik: 2R > 1 > 3 > 4 > 5)
7. **Mega payload preview** — `/admin/staging/preview` ile validate (1744/1744 planned beklenir)
8. **TEK commit + TEK re-embed** — §16 Adım 1-8 yürütme sırası
   - Pre-commit pg_dump backup
   - Commit (atomic)
   - JC0101 manual DELETE (Supabase SQL Editor)
   - regenerate-search-text.ts script (yazılacak, ~30 satır)
   - product_embeddings TRUNCATE + embed-products.ts (~2 dakika)
   - slotExtractor.ts ~25 yeni pattern entry
   - Microservice restart
   - Post-commit doğrulama (taxonomy + smoke test)
   - Bot instruction güncelleme

## Kritik dosyalar (Faz 1 için)

| Dosya | Rol |
|---|---|
| `data/consolidation/phase1-A-volume-payload.json` | Volume/capacity migration (~480 change) |
| `data/consolidation/phase1-B-durability-payload.json` | Durability migration (~50 change) |
| `data/consolidation/phase1-C-ph-payload.json` | pH migration (~80 change) |
| `data/consolidation/phase1-D-dilution-payload.json` | Dilution nested object (~30 change) |
| `data/consolidation/phase1-E-consumption-payload.json` | Consumption + defaults seed (~70 change) |
| `data/consolidation/phase1-F-substrate-payload.json` | target_surface/compatibility/substrate_safe (~25 change) |
| `data/consolidation/phase1-G-specs-subtype-drop-payload.json` | specs.sub_type drop (511 change) |
| `data/consolidation/MEGA-payload.json` | Birleştirilmiş ~1744 change tek payload |
| `retrieval-service/scripts/regenerate-search-text.ts` | search_text yenileme scripti (yazılacak) |
| `Botpress/detailagent-ms/src/conversations/index.ts` | Yeni canonical key listesi + seramik consumption notu |

---

# §18. FINAL YÜRÜTME — 2 commit + doğru sıralama (kullanıcı onayı)

## Context

Sample SKU'lar onaylandı, Faz 1 hazır (924 change), Faz 2R hazır (325), Faz 3R/4/5 mevcut (169). Toplam 1418 change. Kullanıcı kararı: **2 commit, risk düşük tut.**

**Neden 2 commit?** Q2-TYA500M gibi taxonomy değişen SKU'lar (ceramic_coating → tire_care): Faz 1 script'leri MEVCUT DB state'ine göre üretildi. Tek commit'te 30-50 SKU yanlış default alır. 2 commit'te Faz 1 taze DB'ye göre yeniden üretilir.

**Re-embed sıralaması (kullanıcı onayı):** slotExtractor önce, sonra re-embed. Re-embed DB'ye yazar restart gerektirmez, slotExtractor kod değişikliği restart şart.

## 15 adımlı yürütme sırası

| # | Adım | Süre | Komut |
|---:|---|---|---|
| 1 | **pg_dump backup** (3 katmanlı yedek) | ~30 sn | `pg_dump $DB_URL -t products -t product_meta -t product_faqs -t product_relations -t product_search -t product_embeddings > _backup-pre-2R.sql` |
| 2 | **Commit 1: Faz 2R FINAL** (325 change taxonomy) | ~5 sn | `curl POST /admin/staging/commit -d @phase2R-FINAL-payload.json` |
| 3 | **Faz 1 script'lerini yeniden çalıştır** (taze DB ile) | ~30 sn | `bun run scripts/build-phase1-{A,B,C,D,E,F,G}.ts` |
| 4 | **Faz 3R/4/5 uyumluluk kontrolü** | ~1 dk | preview API ile her birini test et |
| 5 | **MEGA payload birleştirme** (Faz 1 + 3R + 4 + 5) | ~10 sn | `scripts/build-mega-payload.ts` |
| 6 | **MEGA preview validation** | ~5 sn | beklenen ~1090/1090 planned |
| 7 | **pg_dump backup #2** (Commit 2 öncesi) | ~30 sn | rollback güvencesi |
| 8 | **Commit 2: MEGA payload** | ~5 sn | atomic transaction |
| 9 | **JC0101 manual DELETE** (Supabase SQL Editor) | ~30 sn | kullanıcı yapar, `phase2R-EXTRAS-sql.md` §1 |
| 10 | **slotExtractor.ts güncelle** (~25 yeni pattern) | ~5 dk | kod değişikliği |
| 11 | **Microservice restart** (slotExtractor aktif) | ~5 sn | `pkill + bun run src/server.ts &` |
| 12 | **Smoke test #1** (slotExtractor pattern doğrulama) | ~2 dk | "yıkama eldiveni" → wash_mitt vb. |
| 13 | **regenerate-search-text + re-embed** | ~2.5 dk | TRUNCATE + embed-products.ts |
| 14 | **Smoke test #2** (vector + BM25 yeni embed) | ~3 dk | searchByQuery sample sorgular |
| 15 | **Bot instruction güncelleme + Smoke test #3** | ~30 dk | yeni canonical key listesi, eval suite |

**Toplam yürütme süresi: ~50 dakika** (büyük kısmı bot instruction güncelleme + smoke test).

## Doğrulama kriterleri (done tanımı)

- [ ] Commit 1 response: `applied:325, errors:0`
- [ ] Commit 2 response: `applied:~1090, errors:0`
- [ ] Post-commit taxonomy: 25 group (spare_part yok), wash_tools var, 511 ürün
- [ ] specs.sub_type tüm ürünlerde silindi (164 → 0)
- [ ] capacity_liters/volume_kg/safe_on_X gibi alias'lar tüm ürünlerde silindi
- [ ] consumption_per_motorcycle_ml yok (kullanıcı uydurması iptal edildi)
- [ ] slotExtractor smoke: wash_mitt, backing_plate, tire_dressing, polish, fabric_leather_cleaner doğru sub_type döndürüyor
- [ ] Vector search: "yıkama eldiveni" → wash_tools sub'larından döndürür (yeni cluster)
- [ ] Admin UI: yeni taxonomy, yeni canonical specs key'leri görünür
- [ ] JC0101 4 tabloda 0 satır
- [ ] Bot instruction: yeni canonical key listesi + seramik 25/15 ml kuralı + eval bir kez geçti

## Rollback senaryoları

| Senaryo | Geri dönüş yolu | Süre |
|---|---|---|
| Commit 1 başarısız | Atomic, otomatik rollback | anında |
| Commit 1 OK ama 2R bozuk | `psql $DB_URL < _backup-pre-2R.sql` | ~2 dk |
| Commit 2 başarısız | Atomic, sadece Commit 1 kaldı (taxonomy hâlâ uygulı) | anında |
| Commit 2 OK ama MEGA bozuk | `psql $DB_URL < _backup-pre-mega.sql` (Adım 7'den) | ~2 dk |
| Re-embed bozuk | EMBEDDING_VERSION eski değere çevir, eski embed'ler kalır | anında |
| Bot instruction bozuk | Git revert (Botpress/detailagent-ms commit) | ~1 dk |

## Önemli not — Production durumu

⚠ **Kullanıcı doğrulamalı:** Bot şu an production'da MI? Re-embed sırasında (~2 dk) vector search kısmen offline. Production'daysa:
- Sabah trafiği düşükken yap (örn. 06:00-07:00)
- VEYA migration'ı staging'de test et önce
- Bu plan kullanıcı LOCAL'de + DB Supabase test/prod ise: prod'da olmadığını teyit et

---

# §19. Phase 2 düzeltmeler — kullanıcı feedback (post-migration audit)

## Context

Migration tamamlandıktan sonra kullanıcı admin UI'da gözden geçirip **9 düzeltme/iyileştirme** önerdi. Bu §19 sadece bunların **silme alt kümesini** (kullanıcı tarafından net onaylanan kısmı) kapsar. Kalan 8 madde için ayrı karar bekleniyor.

## Onaylanan: 7 ürün cascade DELETE

Kullanıcı net silme istedi:

| SKU | Ad | Mevcut grup/sub | FAQ | Meta | Relations |
|---|---|---|---:|---:|---:|
| 20144.261.001 | MENZERNA Likit Matlaştırıcı Pasta 1 lt | abrasive_polish/heavy_cut_compound | 5 | 5 | varsa |
| Q2-BS30M | GYEON Q Booster Seramik Yenileyici 30 ml | ceramic_coating/paint_coating | **19** | 3 | 1 src |
| 700405 | INNOVACAR SINH HYBRID Seramik Kaplama 30ml Set | ceramic_coating/paint_coating | 4 | 4 | **12 src** |
| 532579 | FLEX HG 650 Sıcak Hava Tabancası 2000W | accessory/heat_gun | 3 | 2 | — |
| 530537 | FLEX BW Cordless Blower Hava Üfleyici | accessory/blower | 3 | 1 | 3 src |
| 513547-01 | FLEX Şarjlı Vidalama+Fırça Seti (Matkap+3lü Fırça) | accessory/tool_kit | 3 | 1 | 1 src |
| SGGD402 | SGCB Bahçe Sulama Tabancası 10 Desenli | accessory/water_spray_gun | 3 | 2 | — |

**Toplam etkilenen satır:**
- products: 7 silinir
- product_faqs: ~40 satır silinir
- product_meta: ~18 satır silinir
- product_relations: ~17 satır silinir (sku source) + related_sku olarak geçenler

**Yan etki — accessory grubu:** 4 ürün silinince (532579, 530537, 513547-01, SGGD402), accessory grubunda 5 ürün kalır:
- SGGC086 (air_blow_gun)
- SGGS003 (air_blow_gun)
- SGYC010 (repair_part — Tornador yedek boncuk)
- SGYC011 (repair_part — Tornador yedek hortum)
- SGGC055 (tornador_gun)

→ Sadece **hava tabancası + Tornador ekosistemi** kalır. Grup ismi `accessory` artık daha az anlamlı (madde 4'teki kullanıcı endişesi geçerlilik kazanır — ayrı bir karar).

## Yürütme — atomic SQL transaction (psql)

```sql
BEGIN;

-- Silinecek SKU listesi
WITH sku_list AS (
  SELECT unnest(ARRAY['20144.261.001','Q2-BS30M','700405','532579','530537','513547-01','SGGD402']) AS sku
)

-- 1. Relations: hem source hem target olarak geçenler
DELETE FROM product_relations
WHERE sku IN (SELECT sku FROM sku_list)
   OR related_sku IN (SELECT sku FROM sku_list);

-- 2. FAQs
DELETE FROM product_faqs WHERE sku IN (SELECT sku FROM sku_list);

-- 3. Meta
DELETE FROM product_meta WHERE sku IN (SELECT sku FROM sku_list);

-- 4. Search index + embeddings (foreign key cascade ile gider ama explicit emin olalım)
DELETE FROM product_search WHERE sku IN (SELECT sku FROM sku_list);
DELETE FROM product_embeddings WHERE sku IN (SELECT sku FROM sku_list);

-- 5. Products
DELETE FROM products WHERE sku IN (SELECT sku FROM sku_list);

-- Doğrulama (hepsi 0 olmalı)
SELECT 'products' AS tbl, COUNT(*) FROM products WHERE sku = ANY(ARRAY['20144.261.001','Q2-BS30M','700405','532579','530537','513547-01','SGGD402'])
UNION ALL SELECT 'product_meta', COUNT(*) FROM product_meta WHERE sku = ANY(ARRAY['20144.261.001','Q2-BS30M','700405','532579','530537','513547-01','SGGD402'])
UNION ALL SELECT 'product_faqs', COUNT(*) FROM product_faqs WHERE sku = ANY(ARRAY['20144.261.001','Q2-BS30M','700405','532579','530537','513547-01','SGGD402'])
UNION ALL SELECT 'product_relations', COUNT(*) FROM product_relations WHERE sku = ANY(ARRAY['20144.261.001','Q2-BS30M','700405','532579','530537','513547-01','SGGD402']) OR related_sku = ANY(ARRAY['20144.261.001','Q2-BS30M','700405','532579','530537','513547-01','SGGD402'])
UNION ALL SELECT 'product_search', COUNT(*) FROM product_search WHERE sku = ANY(ARRAY['20144.261.001','Q2-BS30M','700405','532579','530537','513547-01','SGGD402'])
UNION ALL SELECT 'product_embeddings', COUNT(*) FROM product_embeddings WHERE sku = ANY(ARRAY['20144.261.001','Q2-BS30M','700405','532579','530537','513547-01','SGGD402']);

COMMIT;
```

**Beklenen:** Tüm count'lar 0. Atomic transaction → fail olursa rollback otomatik.

## Pre-cascade backup (rollback güvencesi)

```bash
export PATH="/opt/homebrew/opt/libpq/bin:$PATH"
DB_URL=$(grep SUPABASE_DB_URL retrieval-service/.env | cut -d= -f2-)
TS=$(date +%Y%m%d-%H%M%S)
mkdir -p data/consolidation/_pre-7sku-delete-$TS

# Sadece 7 SKU için snapshot al
psql "$DB_URL" --csv -c "
SELECT 'products' AS tbl, row_to_json(p) AS data FROM products p WHERE sku = ANY(ARRAY['20144.261.001','Q2-BS30M','700405','532579','530537','513547-01','SGGD402'])
UNION ALL SELECT 'product_faqs', row_to_json(f) FROM product_faqs f WHERE sku = ANY(...)
-- vb.
" > data/consolidation/_pre-7sku-delete-$TS/sku_snapshot.csv
```

## DB final durum (silme sonrası)

- **Önce:** 510 ürün
- **Sonra:** 503 ürün
- **accessory grubu:** 9 → 5 ürün
- **abrasive_polish:** 23 → 22 ürün (heavy_cut_compound 1 azaldı)
- **ceramic_coating:** 19 → 17 ürün (paint_coating 2 azaldı)

## Karar bekleyen kalan 8 madde (§19'un alt-bölümü değil, ayrı seans)

Aşağıdaki maddeler hâlâ kullanıcı kararı bekliyor (silme sonrası ayrı işlem):

1. ✅ **NPMW6555 → wool_pad** (1 staging change) — onay bekliyor
2. ✅ **industrial_products / metal_polish → solid_compound** (~33 change, surface array + purpose) — onay bekliyor
3. ⚠ **accessory grup ismi** — silme sonrası 5 ürün kalıyor, durum değişti, yeniden değerlendirilmeli
4. ✅ **tool_kit → power_drill_kit** — silme sonrası irrelevant (tool_kit ürünü kalmadı)
5. ✅ **repair_part → tornador_part** (2 staging change) — onay bekliyor
6. ✅ **marin/interior_detailer rename** (4 staging change) — onay bekliyor
7. ✅ **marin/iron_remover + water_spot_remover yeniden adlandır** (3 staging change) — onay bekliyor
8. ✅ **Cross-group isim çakışması — genel kural** (bot instruction güncelleme) — onay bekliyor

Madde 4 (tool_kit) silme ile çözüldü → kapanır.
Madde 3 (accessory) silme sonrası kapsam değişti → tekrar değerlendirilmeli.

Toplam silme sonrası iş: ~43 staging change (NPMW + solid_compound + repair_part + marin sub_type rename) + bot instruction güncelleme.

## Yürütme sırası

1. Pre-cascade backup (psql snapshot)
2. Cascade DELETE transaction (atomic)
3. product_search + product_embeddings'ten 7 SKU silindi (transaction içinde)
4. Doğrulama (count check)
5. Microservice cache temizleme: gerek yok (DB değişti, restart gerekmez, cache yok)

---

# §20. Bot E2E Test Sırası — ADK Webchat (kullanıcı manuel test)

## Context

Phase 18+19+post-Phase 19 tüm kritik fix'ler uygulandı (TEMPLATE_GROUPS enum, specs→product_meta projection, bot instruction güçlendirme). Microservice + ADK dev hazır. Sıra: kullanıcı **webchat'te 10 test sorusunu deneyip sonuçları raporlamak**.

## Mevcut çalışan servisler

- **Microservice:** `localhost:8787` (PID önceden başlatıldı, Phase 19 fix'ler aktif)
- **ADK dev:** `localhost:3000` + tunnel `https://tunnel.botpress.cloud/ad072e5a-e453-4c3b-95c8-52551de6b9da`
- **Bot:** Dev Bot deployed, webchat + chat integration'ları aktif

## Test sırası (önceden hazırlanan §"10 soru" listesinden)

Kullanıcı webchat'i açar, sırayla 10 soruyu (T1-T11) dener. Her test için:
1. **Sorgu** yazılır
2. Bot cevap üretir (carousel + metin)
3. Kullanıcı **doğru/yanlış değerlendirir**, conv_id kaydeder
4. Hatalı olanlar bana raporlanır

## Hata raporlama formatı (kullanıcıdan beklenen)

```
T<n> — conv_id_xxx
Beklenen: <X grup, Y sub>
Geldi: <ne geldi>
Sorun: <bot LLM mi, microservice mi, veri mi>
```

## Düzeltme akışı (her hata için)

1. **Microservice tarafı doğru mu?** Direkt curl ile `/search` test et
2. **product_meta'da key var mı?** psql ile kontrol
3. **slotExtractor pattern doğru mu?** Inline test
4. **Bot LLM tool çağrısı doğru mu?** ADK trace MCP ile incele (ADK_SPAN_INGEST_URL=http://localhost:56513)
5. Düzeltme uygula → bot instruction veya slotExtractor → ADK build → test retry

## Critical files (test sırasında muhtemelen güncellenecek)

- `Botpress/detailagent-ms/src/conversations/index.ts` — instruction
- `Botpress/detailagent-ms/src/tools/search-products.ts` — tool description
- `retrieval-service/src/lib/slotExtractor.ts` — Türkçe pattern
- `retrieval-service/scripts/project-specs-to-meta.ts` — projection (yeni key eklenirse re-run)

## Doğrulama kriteri (done)

- [ ] T1-T11 hepsi doğru sonuç (10/10 — daha önce 6/10 idi)
- [ ] Anti-hallucination: bot uydurma ürün önermiyor
- [ ] Multi-volume: bot doğru boyut filter yapıyor
- [ ] Cross-group: yeni sub_type ve isim çakışması olmadan doğru filter
6. Smoke test: silinen 7 SKU artık search sonucunda görünmemeli
