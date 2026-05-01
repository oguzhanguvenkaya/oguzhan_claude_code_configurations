# Innovacar Blog Verisini Eşdeğer Ürün Haritalaması ile Projeye Entegre Etme

## Context

Innovacar'ın 5 detailing blog rehberini (`docs/innovacar-blogs-scraped.md`) kazıdık. Bloglar; **yıkama → dekontaminasyon → polisaj → glaze → koruma → bakım** olmak üzere tam bir detailing akışını sıralı ürün adımlarıyla anlatıyor. Katalogumuzda bloglardaki bazı Innovacar ürünleri yok (D3 TAR, F1/A1/P1 cila üçlüsü, G1 GLOSSY, SINH TOP COAT, clay hattı, Micron bezleri, vb.).

**Problem:** Blog içerikleri dağınık markdown dosyasında duruyor; bot bu prosedürel bilgiyi sorgulayamıyor. Ayrıca blog akışındaki eksik ürünler "unknown SKU" olarak kaldığı için relations graph'ımıza yansımıyor.

**İstenen sonuç:** Blog'daki eksik ürünleri katalogdaki **eşdeğer SKU'lara haritalayarak**, prosedürel içeriği mevcut tablolara (productContentTable, productRelationsTable, productFaqTable) enjekte etmek. Böylece bot, marka bağımsız detailing prosedürlerini cevaplayabilsin ve "tam detailing nasıl yapılır" gibi çok-aşamalı soruları tutarlı bir şekilde yanıtlasın.

---

## Mimari Yaklaşım

### 1. Eşdeğerlik Haritası (`data/innovacar-equivalents.csv`)

Blogda geçen ama katalogta olmayan her ürün için **en yakın fonksiyonel eşdeğeri** (marka bağımsız) bulunur. Bu tablo `scripts/map-innovacar-equivalents.ts` tarafından tüketilir.

```csv
innovacar_product, category, equivalent_skus, confidence, notes
D3 TAR, tar_remover, <mts_tar_sku>, high, zift/katran sökücü
F1 FORCE ONE, heavy_cut_compound, 22746.281.001;22200.281.001, high, Menzerna 300/400 eşdeğer
A1 ALL IN ONE, medium_cut_polish, <menzerna_2500_sku>, high, orta kesim cila
P1 POLISH UP, finish_polish, <menzerna_3800_sku>, high, finish cila
G1 GLOSSY, glaze, <glaze_sku_or_null>, medium, glaze kategorisi; yoksa "not_stocked"
SINH TOP COAT, ceramic_coating, 700405, medium, SINH HYBRID yakın eşdeğer
CLAY BAR, clay_bar, <clay_sku>, high, mekanik dekontaminasyon
DL LUBE, clay_lube, <clay_lube_sku>, medium
MICRON *, microfiber_cloth, <generic_mf_sku>, low, aksesuar — opsiyonel atla
```

Eşdeğerlik `template_group` + `template_sub_type` benzerliğine göre doğrulanır (products_master.csv sütunları).

### 2. Veri Katmanı Enjekte Noktaları

Mevcut tablolara **yeni tablo açmadan** yazılır:

| Blog Çıktısı | Hedef Tablo | Nasıl |
|---|---|---|
| 5 faz akışı (yıkama→bakım) | `productRelationsTable.use_before / use_after` | Her eşdeğer SKU için önceki/sonraki fazdaki SKU'lar graph'a eklenir. Mevcut `scripts/upsert-relations-paket-i.ts` kalıbı kullanılır. |
| Fazlardaki birlikte kullanım (ör. clay + lube) | `productRelationsTable.use_with` | Aynı fazda geçen ürünler `use_with` olarak linklenir. |
| Ürün spesifik kullanım ("SC0 1 saat kurumalı") | `productContentTable.howToUse` (6-adım formatı) | Blog'daki sırayla adımlara dönüştürülür; mevcut 6-adım şemasına uyarlanır. |
| Kategori-seviyesi rehber ("glaze nedir, ne işe yarar") | `productFaqTable` — `_CAT:<group>` konvansiyonu | Mevcut `scripts/seed-category-faqs.ts` kalıbı. Örn: `_CAT:glaze`, `_CAT:sealant_vs_ceramic`, `_CAT:decontamination`. |
| Teknik özellikler ("sıcaklık +5/+25°C", "1 saat cure") | `productMetaTable` (EAV) | `application_temp_min`, `application_temp_max`, `cure_time_hours` gibi key'lerle. Mevcut `scripts/seed-meta-table.ts` kalıbı. |

### 3. Script Sırası

Değiştirilecek/oluşturulacak dosyalar:

1. **YENİ:** `data/innovacar-equivalents.csv` — eşdeğerlik tablosu (elle doldurulur, bir kereliğine)
2. **YENİ:** `data/innovacar-procedures.csv` — blog'daki 5 faz × adım × SKU matrisi
3. **YENİ:** `data/innovacar-category-faqs.csv` — kategori seviyesi Q&A (glaze, sealant, decon)
4. **YENİ:** `scripts/seed-innovacar-procedures.ts` — `upsert-relations-paket-i.ts` kalıbı
   - Blog sırasından `use_before/after/with` türet
   - Mevcut relations'a **merge** et (üstüne yazma, dedup)
5. **YENİ:** `scripts/seed-innovacar-category-faqs.ts` — `seed-category-faqs.ts` kalıbı
6. **YENİ:** `scripts/seed-innovacar-meta.ts` — teknik özellikleri productMetaTable'a
7. **GÜNCELLEME:** `src/conversations/index.ts` içindeki "SET/PAKET workflow" (satır ~175-202) bölümü, yeni kategori FAQ'larına (ör. `_CAT:glaze`) referans verecek şekilde küçük bir ekleme alabilir.

### 4. Doğrulama / Test

1. **Build & seed:**
   ```bash
   bun run scripts/seed-innovacar-procedures.ts
   bun run scripts/seed-innovacar-category-faqs.ts
   bun run scripts/seed-innovacar-meta.ts
   adk build
   ```
2. **MCP `adk_send_message` ile end-to-end sorular:**
   - "Detaylı bir full detailing sürecini anlat" → 5 faz + ürün sırası gelmeli
   - "Glaze nedir, ne zaman uygulanır?" → `_CAT:glaze` FAQ'sı tetiklenmeli
   - "Sealant ve seramik arasındaki fark" → `_CAT:sealant_vs_ceramic` tetiklenmeli
   - "Clay bar sonrası ne yapmalıyım" → `use_after` relations graph'ından cevap
3. **Evals:** `evals/` altında en az 5 yeni soru ekle (category workflow soruları).
4. **Relations dedup kontrol:** Seed sonrası `productRelationsTable`'da aynı (sku, target_sku) çifti birden fazla satırda olmamalı.

---

## Bot Cevap Kalitesine Etkisi

| Boyut | Mevcut Durum | Entegrasyon Sonrası |
|---|---|---|
| **Prosedürel bilgi** | Tekil ürün howToUse'u var; faz-arası akış zayıf | 5 fazlı tam akış `use_before/after` üzerinden traverse edilebilir |
| **Kategori bilgisi** | `_CAT:<group>` FAQ'ları mevcut ama eksik kategoriler var (glaze, decon, sealant_vs_ceramic) | Blog özetlerinden türetilmiş kategori-seviyesi açıklama eklenir |
| **"Tam set öner"** akışı | `conversations/index.ts` SET/PAKET flow'u var; ürün seçimi manuel | Blog akışı referans alınarak otomatik paket önerisi (yıkama+decon+cila+koruma) tutarlılaşır |
| **Marka bağımsızlık** | Marka-spesifik cevaplar | Innovacar akışı Menzerna/diğer eşdeğerlerle tercüme edilir → marka-agnostik öneri |
| **Teknik parametre** | Sıcaklık/cure gibi parametreler serbest metinde | `productMetaTable` EAV üzerinden filtrelenebilir ("+5°C altında uygulanabilir seramik") |
| **Hallucination riski** | Bloglardaki bilgiyi bot uyduramayabilir | Yapısal tablolara inince kaynak-doğrulanabilir cevap |

**Beklenen somut iyileşmeler:**
- Çok-adımlı ("nasıl yapılır") sorularda recall artışı
- Eksik fazlı cevaplarda (ör. cila'yı atlayan öneriler) tutarlılık
- "Ne zaman glaze kullanılır" gibi kategori sorularında cevap olgunluğu
- Relations graph yoğunluğu arttığı için `get-related-products` tool çağrılarının faydalı çıktı oranı yükselir

**Riskler:**
- Yanlış eşdeğerlik haritası → bot yanlış ürün önerir (mitigation: `confidence` kolonu + elle review)
- Blog içeriğinin İtalyan piyasası bağlamı Türkiye koşullarıyla tam örtüşmeyebilir (osmoz suyu, +160 bar vb.) → FAQ cevaplarında lokalize et
- Micron/aksesuar hattındaki birebir eşdeğer zor (farklı markalar) → düşük confidence olanları `not_stocked` olarak işaretleyip hariç tut

---

## Kritik Dosyalar (Referans)

- `src/tables/product-relations.ts` — use_before/after/with şeması
- `src/tables/product-content.ts` — howToUse 6-adım formatı
- `src/tables/product-faq.ts` — `_CAT:<group>` konvansiyonu
- `src/tables/product-meta.ts` — EAV şeması
- `scripts/upsert-relations-paket-i.ts` — relations seed kalıbı (14cec6c commit)
- `scripts/seed-category-faqs.ts` — kategori FAQ seed kalıbı
- `scripts/seed-meta-table.ts` — EAV seed kalıbı
- `src/conversations/index.ts:175-202` — SET/PAKET workflow (güncelleme noktası)
- `docs/innovacar-blogs-scraped.md` — kaynak blog kazıma çıktısı
- `/Users/projectx/Desktop/Claude Code Projects/Products Jsons/output/csv/products_master.csv` — eşdeğer aday SKU kaynağı

---

## Önerilen Aşamalı Uygulama

1. **Faz A (eşdeğerlik):** `data/innovacar-equivalents.csv` elle hazırla — en kritik 10-12 ürün için
2. **Faz B (prosedür):** `innovacar-procedures.csv` + seed script → relations graph zenginleştir
3. **Faz C (kategori FAQ):** 4-6 yeni `_CAT:*` kaydı → kategori-seviyesi cevap kalitesi
4. **Faz D (meta):** Teknik parametreleri EAV'a ekle (opsiyonel, son aşama)
5. **Faz E (eval):** Yeni eval case'leri ekle ve run et — regresyon kontrolü
