# GYEON Chatbot — Kapsamlı Search & Routing Fix Paketi v9.1

## Context

25 soruluk test sonucu bot'ta 6 core search/routing problemi tespit edildi. Çözüm **küçük-adım, her-aşamada-test** şeklinde uygulanacak. Genel kapsayıcı kurallar — her soruya özel prompt değil. Git push her adımda.

**Kritik bulgu:** Bot #12'de (**CanCoat EVO mat boya**) veri ile TERS cevap verdi:
- Bot: "mat yüzeylere uygun, parlatmaz" ❌
- Gerçek FAQ (satır 252): "Evet parlaklık sağlar, mat yüzeyler için uygun değildir" ✓

Bu, "sayısal değil metin içerikli FAQ" durumunda SPEC-FIRST kuralının eksik olduğunu gösteriyor.

---

## Tespit Edilen 6 Core Problem

### P1: exactMatch 0 sonuç → Semantik fallback YOK
- **Kanıt:** "FabriCoat" yazımı → `exactMatch="FabriCoat"` → regex `\bFabriCoat` hiçbir şey eşlemiyor (master "FabricCoat") → 0 sonuç
- **Kök:** Regex path'ten `search:` semantik path'e fallback yok
- **Etki:** 1 harf fark veya yazım varyasyonlarında ürün bulunamıyor

### P2: exactMatch Multi-Match Sıralama
- **Kanıt:** "OdorRemover" → hem `Q2M-OR500M` (sprey) hem `Q2M-ORP4P` (pads) eşleşiyor
- **Kök:** DB natural order kullanılıyor, similarity/relevance yok
- **Etki:** Kullanıcı "Pads" demedi mi → yanlış varyant gelebiliyor

### P3: searchFaq Context-Free
- **Kanıt:** "Pure EVO 2 kat uygulanır mı?" → Compound FAQ (Q2M-CM1000M) döndü, Pure EVO FAQ'ı değil
- **Kök:** FAQ aramada ürün filter yok, tüm 2413 FAQ'yi tarıyor
- **Etki:** Ürün-spesifik sorularda yanlış ürünün FAQ'ı sunuluyor

### P4: Bot `confidence='low'` FAQ'ı yine sunuyor
- **Kanıt:** #12 CanCoat EVO mat sorusu → topSim 0.523 (low) → bot yine "evet kullanılır" dedi (hatta veri ters)
- **Kök:** Instruction low durumunda "disclaimer ile sun" diyor, bot uydurmaya yakın yanıt verebiliyor
- **Etki:** Yanlış bilgi + uydurma riski

### P5: Ratings Karşılaştırma — Sınırlı Tarama
- **Kanıt:** "Self-cleaning en iyi seramik?" → sadece 5 ürün getProductDetails, 8 SC=5.0 ürünü var (Mohs, Pure, View, Matte, Rim, Trim, PPF, FabricCoat)
- **Kök:** Bot searchProducts default limit=5, ratings karşılaştırması için batch arama yok
- **Etki:** Yanlış veya eksik öneri

### P6: template_group Filter Yanlış Uygulanıyor
- **Kanıt:** LeatherShield EVO `templateGroup="leather_care"` filter → 0 sonuç. Gerçek: `template_group="ceramic_coating", sub_type="leather_coating"`
- **Kök:** Bot veri bilgisi eksik, filter seçimi spekülatif
- **Etki:** Tanınan ürünler bulunamıyor

---

## Fix Paketi (küçük adımlar, her adım test)

### Aşama A: GitHub'a Önce Push (güvenlik)
Mevcut durumu korumak için, fix'lerden ÖNCE master'a push:
```bash
cd "/Users/projectx/Desktop/Claude Code Projects/Products Jsons"
git add -A
git commit -m "chore: v9.0 pre-fix snapshot (25-question test + Faz 3 enrichment complete)"
git push origin main
```

### Aşama B: Fix B1 — exactMatch → Semantic Fallback (P1)

**Dosya:** `src/tools/search-products.ts`

exactMatch post-filter + broad-fallback 0 döndüyse, `search: query` parametresi ile semantik arama yap:

```ts
if (fetchResult.rows.length === 0) {
  // Semantic fallback — yazım farkları ve yakın eşleşmeler için
  const semanticRes = await client.findTableRows({
    table: 'productSearchIndexTable',
    search: query,  // query parametresi (semantic vector search)
    filter: Object.keys(filter).length > 0 ? filter : undefined,
    limit,
  });
  fetchResult = semanticRes;
}
```

**Test:** "FabriCoat" → FabricCoat bulmalı

### Aşama C: Fix B2 — searchFaq SKU-aware + Low Confidence Stricter (P3, P4)

**Dosya:** `src/tools/search-faq.ts`

İki değişiklik:
1. Opsiyonel `sku` parametresi → verildiyse filter
2. `confidence='low'` + metin sorusu (sayısal değil) → results boş (özgür davranışa izin verme)

```ts
input: z.object({
  query: z.string(),
  sku: z.string().optional().describe('Spesifik ürün SKU (context-aware filter)'),
  limit: z.number().int().min(1).max(10).default(5),
}),

// handler'da
const filter: Record<string, any> = {};
if (sku) filter.sku = { $eq: sku };
const res = await client.findTableRows({
  table: 'productFaqTable',
  search: query,
  filter: Object.keys(filter).length > 0 ? filter : undefined,
  limit,
});
```

### Aşama D: Fix B3 — Multi-Match Preference (P2)

**Dosya:** `src/tools/search-products.ts`

exactMatch fallback'te multi-match varsa, product_name'de `needle`'den hemen sonra boşluk veya sonu olanları **önce** getir (en spesifik match). Örn "OdorRemover " (boşluk) → sprey; "OdorRemover P..." (yeni kelime başlangıcı) → Pads.

```ts
const postFiltered = broadRes.rows.filter(...).sort((a, b) => {
  const nameA = String(a.product_name || '').toLowerCase();
  const nameB = String(b.product_name || '').toLowerCase();
  const needleLower = exactMatch.toLowerCase();
  // 'OdorRemover ' ya da 'OdorRemover-' → prefix match score yüksek
  // 'OdorRemover Pads' → needle sonrası var ama ekstra kelime (daha uzun name)
  return nameA.length - nameB.length;  // Daha kısa = daha generic match
});
```

Bu, **kullanıcı daha spesifik demediyse** varsayılan olarak en generic ürünü dönmesini sağlar.

### Aşama E: Fix B4 — Instruction Güncellemesi (P3, P4, P5, P6)

**Dosya:** `src/conversations/index.ts`

Yeni kapsamlı kurallar (her duruma özel değil, GENEL pattern'lar):

```
## FAQ ARAMA — Context Öncelikli (v9.1)

searchFaq ÇAĞIRMADAN ÖNCE:
- state.lastFocusSku varsa → onu FAQ'a filter olarak geç
- Kullanıcı SPESİFİK ürün adı/SKU belirttiyse → önce searchProducts, SKU'yu al, sonra searchFaq({sku})
- Ürün belirsizse ve soru genel ise → searchFaq filter'sız OK

## DÜŞÜK CONFIDENCE = UYDURMA (v9.1)

searchFaq confidence = low veya none:
- Cevabı ASLA direkt sunma
- "Bu konuda kesin bilgi bulamadım" de
- Alternatif: getProductDetails.faqs'ten ara
- Asla "sanırım", "muhtemelen" + FAQ içeriği sunma

## RATINGS KARŞILAŞTIRMA (v9.1 yeni)

"en iyi X", "en yüksek Y", "performansı en iyi" sorularında:
- searchProducts(templateGroup=ilgili kategori, limit=10+) ile geniş set al
- Her ürün için getProductDetails çağır, technicalSpecs.ratings kontrol et
- ratings alanı olan ürünleri skor'a göre sırala
- Top-3'ü carousel olarak sun

## template_group FILTER (v9.1)

LLM template_group'u KESİN BİLMİYORSA filter'a koyma:
- Örn. "deri koruyucu" → leather_care OLABİLİR veya ceramic_coating/leather_coating
- Önce filter'sız ara, template_group'a göre daraltma post-filter
- Yanlış filter = 0 sonuç riski
```

### Aşama F: Fix B5 — Smoke Test Uygula

Her aşamadan sonra kullanıcı manuel test eder, sonuçları paylaşır.

---

## Kapsamlı Test Suite — 30 Soru (Farklı Markalar)

Yeni dosya: `output/gyeon_official/comprehensive_test_30.md`

### GYEON (12 soru)
1. "FabriCoat kumaş koltuk kaç ay dayanır?" (P1 — yazım varyasyonu)
2. "OdorRemover Pads kaç hafta dayanır?" (P2 — multi-match)
3. "CanCoat EVO mat boyaya uygulanır mı?" (P4 — FAQ güvenilirliği)
4. "Q² Pure EVO iki kat uygulanabilir mi?" (P3 — context)
5. "Self-cleaning performansı en yüksek üç seramik?" (P5 — ratings karşılaştırma)
6. "LeatherShield EVO kürlenme süresi?" (P6 — template_group)
7. "Mohs EVO ile Pure EVO arasındaki fark?"
8. "PPF EVO ile LeatherShield EVO aynı amaç için mi?"
9. "Bathe vs Bathe+ Plus arasındaki fark?"
10. "GYEON pH nötr şampuanları listele"
11. "Wetcoat ıslak araca uygulanabilir mi?"
12. "Q²R Marine ürünleri var mı?" (NO_MATCH markasında davranış)

### MENZERNA (6 soru)
13. "Menzerna 400 hangi pad ile kullanılır?"
14. "Menzerna 3800 finish pasta mıdır?"
15. "Menzerna silikonsuz mu?"
16. "Menzerna kalın pasta seçenekleri"
17. "Menzerna 400 250ml fiyatı?"
18. "Menzerna pasta sırası (compound → polish → finish)?"

### FRA-BER (3 soru)
19. "FRA-BER hangi kategoride ürünler sağlıyor?"
20. "FRA-BER tente temizleyicisi var mı?"
21. "FRA-BER endüstriyel ürünler?"

### Innovacar (2 soru)
22. "Innovacar marka ürünleri listele"
23. "Innovacar'ın otomotiv bakımındaki yeri?"

### Genel & Cross-Brand (7 soru)
24. "Seramik kaplama uygulama sırası (adım adım)?"
25. "3 farklı marka ağır çizik pastası öner"
26. "500 TL altı şampuan?"
27. "1000 TL civarı seramik kaplama?"
28. "Polisaj pedi türleri (foam/wool/microfiber)?"
29. "Kumaş koltuk temizleyici + koruyucu takım?"
30. "Detaylı paket bakım için 5000 TL bütçe önerin?"

---

## Uygulama Sırası (küçük adımlar)

1. **Git push** — mevcut durumu koru
2. **Fix B1** (exactMatch → semantic fallback) — `adk build` doğrula
3. **Kullanıcı test:** P1 soruları (FabriCoat, LeatherShield)
4. **Fix B2** (searchFaq SKU + low confidence)
5. **Kullanıcı test:** P3/P4 soruları (Pure EVO, CanCoat mat)
6. **Fix B3** (multi-match preference)
7. **Kullanıcı test:** P2 soruları (OdorRemover)
8. **Fix B4** (instruction güncellemesi)
9. **Kullanıcı test:** Tüm 30 soru
10. **Git push** — fix'ler commit
11. Production'a deploy (opsiyonel, kullanıcı kararı)

---

## Çıktı Dosyaları

- `Botpress/detailagent/src/tools/search-products.ts` — Fix B1, B3
- `Botpress/detailagent/src/tools/search-faq.ts` — Fix B2
- `Botpress/detailagent/src/conversations/index.ts` — Fix B4
- `output/gyeon_official/comprehensive_test_30.md` — 30-soru test suite
- Git commits: `v9.0 snapshot`, `fix(search): B1 semantic fallback`, ...

## Doğrulama

- `adk build` her fix sonrası temiz
- Manual smoke test: kullanıcı dev bot'ta soru sorar
- Başarı eşiği: 30/30 → %80 doğru cevap (≥24)
- P1/P3/P4 soruları öncelikli — bunlar önceki testte fail oldu
