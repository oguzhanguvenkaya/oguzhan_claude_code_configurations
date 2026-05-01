# Phase 1.1.13D — compatibility split (safe_on + equipment_compat) + substrate_safe merge

## Context

`specs.compatibility` 10 üründe **3 farklı kavram** birden tutuyor — bot için karıştırma riski:

| Kavram | Ürün | Örnek değer |
|---|---|---|
| **A. Yüzey güvenliği** ("üzerinde güvenli") | 5 (şampuanlar) | `["ceramic_coating"]` |
| **B. Folyo materyali** (PPF kurulum) | 2 (PPF Lift/Slip) | `["TPU", "TPH", "Vinil", "Stealth PPF"]` |
| **C. Ekipman uyumu** (marka/seri) | 3 (foam lance, sprayer parça, ragle) | `Karcher K Serisi`, `IK 1.5/2 litre tanklar`, `10 cm standart ragleler` |

`specs.substrate_safe` 2 üründe (`["aluminum","fiberglass","plexiglass"]`, `["aluminum"]`) — semantik olarak A ile aynı (yüzey güvenliği).

**Coverage gerçeği (kullanıcı içgörüsü):** safe_on **sadece kimyasal ürünlerde** (şampuan, koruma, temizleyici) anlamlı. Mekanik aksesuarlarda (pompa, ragle, sprayer parça) bu meta key bulunması saçma — yani 484 üründe "eksik" değil, **anlamsız**. Bu fazda enrichment yapılmıyor; mevcut 12 ürün (10 compat + 2 substrate_safe) elden geçirilir.

**Bot prompt mevcut kullanım (5 yer):**
- [search-products.ts:165-166](Botpress/detailagent-ms/src/tools/search-products.ts#L165) description
- [search-products.ts:197-198](Botpress/detailagent-ms/src/tools/search-products.ts#L197) örnek metaFilter
- [search-products.ts:204, 207](Botpress/detailagent-ms/src/tools/search-products.ts#L204) ARRAY listesi
- [conversations/index.ts:379](Botpress/detailagent-ms/src/conversations/index.ts#L379) "jant temizleyici" notu
- [conversations/index.ts:623-624, 633, 643](Botpress/detailagent-ms/src/conversations/index.ts#L623) örnekler + ARRAY tanımı
- [get-product-details.ts:50-51](Botpress/detailagent-ms/src/tools/get-product-details.ts#L50)

## Kullanıcı kararları (3-soru AskUserQuestion sonrası)

1. **Yaklaşım: Hibrit (2 field)** — `safe_on` + `equipment_compat`. Folyo materyali (TPU/TPH/Vinil/Stealth PPF) `safe_on=['ppf','vinil']`'e merge.
2. **substrate_safe:** `safe_on`'a merge (anlamsal birleşik). specs.substrate_safe SİL.
3. **Veri eksikliği:** Bot prompt'a "coverage düşük" uyarısı **EKLEME** — kullanıcının notu: 484 üründe safe_on olması zaten saçma (mekanik ürünler), filter doğru çalışıyor.

## Aksiyon listesi (deterministik, 12 ürün)

### A. safe_on field doldurma (9 ürün)

#### A.1 — Yüzey güvenliği (compatibility'den, 5 ürün)

| SKU | Eski compatibility | Yeni safe_on |
|---|---|---|
| 700507 | `["ceramic_coating"]` | `["ceramic_coating"]` |
| 71490 | `["ceramic_coating"]` | `["ceramic_coating"]` |
| 79290 | `["ceramic_coating"]` | `["ceramic_coating"]` |
| Q2M-EW1000M | `["ceramic_coating"]` | `["ceramic_coating"]` |
| Q2M-RWYA1000M | `["ceramic_coating"]` | `["ceramic_coating"]` |

#### A.2 — PPF folyo materyali (compatibility'den, 2 ürün)

PPF kurulum sıvıları (PPF Lift/Slip) → "PPF folyo üzerinde güvenli + vinil folyo üzerinde güvenli". TPU/TPH/Stealth PPF tokenları PPF altkümesi → canonical `ppf`'e indirilir, `Vinil` ayrı tokendir.

| SKU | Eski compatibility | Yeni safe_on |
|---|---|---|
| PPF-L500M | `["TPU", "TPH", "Vinil", "Stealth PPF"]` | `["ppf", "vinil"]` |
| Q2M-PPFSL4000M | `["TPU", "TPH", "Vinil"]` | `["ppf", "vinil"]` |

#### A.3 — substrate_safe → safe_on merge (2 ürün)

| SKU | Eski substrate_safe | Yeni safe_on |
|---|---|---|
| 701851 (Camper Wash) | `["aluminum","fiberglass","plexiglass"]` | `["aluminum","fiberglass","plexiglass"]` |
| 75021 (Megafoam) | `["aluminum"]` | `["aluminum"]` |

### B. equipment_compat field doldurma (3 ürün)

Free-text (her marka/seri farklı, canonical enum DEĞİL). String SCALAR; bot regex ile arar.

| SKU | Eski compatibility | Yeni equipment_compat |
|---|---|---|
| SGPM008 | `10 cm standart ragleler` | `10 cm standart ragleler` |
| 81671901 | `IK 1.5/2 litre tanklar` | `IK 1.5/2 litre tanklar` |
| SGGD135 | `Karcher K Serisi` | `Karcher K Serisi` |

### C. specs DELETE'ler

- 10 üründe `specs.compatibility` SİL
- 2 üründe `specs.substrate_safe` SİL

**Net etki:**
- specs.compatibility: -10 row (DELETE)
- specs.substrate_safe: -2 row (DELETE)
- specs.safe_on: +9 row (INSERT, ARRAY pipe-separated)
- specs.equipment_compat: +3 row (INSERT, SCALAR free-text)

**Toplam: 12 ürün etkilenir, 24 row aksiyonu** (12 DELETE + 12 INSERT).

### D. Projector güncelleme

[retrieval-service/scripts/project-specs-to-meta.ts](retrieval-service/scripts/project-specs-to-meta.ts):

```diff
  const SCALAR_KEYS = [
    ...
    'ph_category',
+   // Phase 1.1.13D: equipment_compat free-text (Karcher, IK, vb.) — bot regex match.
+   'equipment_compat',
  ];

- const ARRAY_KEYS = ['target_surfaces', 'compatibility', 'substrate_safe'];
+ const ARRAY_KEYS = ['target_surfaces', 'safe_on'];
+ // Phase 1.1.13D: compatibility 3 kavrama split (safe_on + equipment_compat),
+ // substrate_safe → safe_on'a merge. Eski 'compatibility' ve 'substrate_safe'
+ // STALE_KEYS'te (idempotent re-project).

  const STALE_KEYS = [
    ...
    'ph_category',
+   // Phase 1.1.13D: split + merge (compatibility ve substrate_safe deprecated)
+   'compatibility',
+   'substrate_safe',
+   // safe_on + equipment_compat ilk kez yazılıyor ama re-project için STALE
+   'safe_on',
+   'equipment_compat',
  ];
```

Verify SQL list'inden `compatibility` ve `substrate_safe` çıkar, `safe_on` + `equipment_compat` ekle.

### E. Search-text snippet'e explicit ekle

[retrieval-service/scripts/regenerate-search-text.ts](retrieval-service/scripts/regenerate-search-text.ts):

`Object.entries.slice(0,8)` sırasına güvenmiyoruz (Phase 1.1.13C paterni). pH bloğunun yanına ekle:

```ts
// Phase 1.1.13D: safe_on + equipment_compat explicit (kavram netliği için)
const safeOn = specs.safe_on;
if (Array.isArray(safeOn) && safeOn.length > 0) parts.push(`Üzerinde güvenli: ${safeOn.join(', ')}`);
else if (typeof safeOn === 'string' && safeOn.length > 0) parts.push(`Üzerinde güvenli: ${safeOn.replace(/\|/g, ', ')}`);
const eqCompat = specs.equipment_compat;
if (typeof eqCompat === 'string' && eqCompat.length > 0) parts.push(`Ekipman uyumu: ${eqCompat}`);
```

### F. Bot instruction güncelle

[Botpress/detailagent-ms/src/tools/search-products.ts](Botpress/detailagent-ms/src/tools/search-products.ts):

```diff
  "ph_tolerance (string range, kaplama dayanımı), cut_level, " +
  ...
- "compatibility (array: ceramic_coating, ppf — üzerine uygulanabilir), " +
- "substrate_safe (array: aluminum, fiberglass, plexiglass), " +
+ "safe_on (array: ceramic_coating, ppf, aluminum, fiberglass, plexiglass — üzerinde güvenli/zarar vermediği), " +
+ "equipment_compat (string free-text: 'Karcher K Serisi', 'IK 1.5/2 litre tanklar' vb. — marka/seri ekipman uyumu), " +
```

```diff
- "- 'seramik üzerinde güvenli' → [{key:'compatibility', op:'regex', value:'ceramic_coating'}]\n" +
- "- 'alüminyum jant için' → [{key:'substrate_safe', op:'regex', value:'aluminum'}]\n" +
+ "- 'seramik kaplama üzerinde güvenli şampuan' → [{key:'safe_on', op:'regex', value:'ceramic_coating'}]\n" +
+ "- 'PPF folyo üzerinde güvenli' → [{key:'safe_on', op:'regex', value:'ppf'}]\n" +
+ "- 'alüminyum üzerinde güvenli (Camper/karavan şampuanı)' → [{key:'safe_on', op:'regex', value:'aluminum'}]\n" +
+ "- 'Karcher K serisi uyumlu foam lance' → [{key:'equipment_compat', op:'regex', value:'Karcher'}]\n" +
+ "- 'IK 1.5 lt tank uyumlu yedek parça' → [{key:'equipment_compat', op:'regex', value:'IK'}]\n" +
```

```diff
- "**ARRAY key listesi (op:'regex' kullan):** target_surfaces, compatibility, substrate_safe\n" +
+ "**ARRAY key listesi (op:'regex' kullan):** target_surfaces, safe_on\n" +
- "**SCALAR key (op:'eq'/'gte'/'lte'):** product_type, purpose, ph_level, ph_category ..., hardness, ph_tolerance\n\n" +
+ "**SCALAR key (op:'eq'/'gte'/'lte'/'regex'):** product_type, purpose, ph_level, ph_category ..., hardness, ph_tolerance, equipment_compat (free-text, 'regex' kullan)\n\n" +
- "Array key'lerde (target_surfaces, compatibility, substrate_safe) op:'regex' kullan\n"
+ "Array key'lerde (target_surfaces, safe_on) op:'regex' kullan; equipment_compat SCALAR ama free-text → 'regex' kullan.\n"
```

[Botpress/detailagent-ms/src/conversations/index.ts](Botpress/detailagent-ms/src/conversations/index.ts):

- **Satır 379** (jant temizleyici notu): `substrate_safe regex aluminum` → `safe_on regex 'aluminum'`
- **Satır 623** (seramik üzerinde güvenli): `compatibility regex 'ceramic_coating'` → `safe_on regex 'ceramic_coating'`
- **Satır 624** (alüminyum/fiberglass): `substrate_safe regex 'aluminum'` → `safe_on regex 'aluminum'`
- **Satır 633** (ARRAY key listesi): `target_surfaces, compatibility, substrate_safe` → `target_surfaces, safe_on`
- **Satır 643** (ARRAY açıklama): `target_surfaces / compatibility / substrate_safe` → `target_surfaces / safe_on`

[Botpress/detailagent-ms/src/tools/get-product-details.ts:50-51](Botpress/detailagent-ms/src/tools/get-product-details.ts#L50):

```diff
- "compatibility (array: ceramic_coating, ppf — üzerine uygulanabilir), " +
- "substrate_safe (array: aluminum, fiberglass, plexiglass — zarar vermediği), " +
+ "safe_on (array: ceramic_coating, ppf, vinil, aluminum, fiberglass, plexiglass — üzerinde güvenli/zarar vermediği yüzeyler — Phase 1.1.13D, eski compatibility+substrate_safe birleşik), " +
+ "equipment_compat (string free-text: 'Karcher K Serisi' vb. — marka/seri ekipman uyumu, foam lance/sprayer parça için), " +
```

## Implementation Steps

### Faz A — Migration script (~15 dk)

[retrieval-service/scripts/phase-1-1-13d-compat-split.ts](retrieval-service/scripts/phase-1-1-13d-compat-split.ts) (yeni):

```ts
// İki mod: audit (default) + --apply (transaction)
// Hardcoded MAPPING: 12 SKU × { safe_on: string[]|null, equipment_compat: string|null }
// Guards:
//   - 10 compatibility ürünü + 2 substrate_safe ürünü = 12 unique SKU
//   - DB'deki mevcut compatibility/substrate_safe değerleri beklenenle eşleşiyor mu
//   - safe_on tokenları canonical: {'ceramic_coating', 'ppf', 'vinil', 'aluminum',
//                                    'fiberglass', 'plexiglass'}
// Transaction:
//   - safe_on INSERT: jsonb_set(specs - 'compatibility' - 'substrate_safe',
//                                '{safe_on}', to_jsonb(<text[]>::jsonb), true)
//   - equipment_compat INSERT: jsonb_set(specs - 'compatibility',
//                                '{equipment_compat}', to_jsonb($::text), true)
// Audit JSON: scripts/audit/_phase-1-1-13d-audit.json
```

### Faz B — Projector + re-project (~5 dk)

```bash
cd retrieval-service && bun run scripts/project-specs-to-meta.ts
```

**Beklenen değişimler:**
- `product_meta.compatibility`: 10 → 0 (STALE)
- `product_meta.substrate_safe`: 2 → 0 (STALE)
- `product_meta.safe_on`: 0 → 9 row (yeni ARRAY)
- `product_meta.equipment_compat`: 0 → 3 row (yeni SCALAR value_text)

### Faz C — Search-text regen + embedding (~5 dk)

```bash
cd retrieval-service
bun run scripts/regenerate-search-text.ts                              # 494 ürün full regen
bun -e "import {sql} from './src/lib/db.ts'; await sql\`UPDATE product_embeddings SET embedding = NULL WHERE sku IN (SELECT sku FROM products WHERE specs ? 'safe_on' OR specs ? 'equipment_compat')\`; process.exit(0)"
bun run scripts/embed-products.ts  # 12 NULL embedding refresh
```

### Faz D — Verification + commit (~10 dk)

| # | Test | Beklenen |
|---|---|---|
| 1 | `SELECT COUNT(*) FROM products WHERE specs ? 'compatibility'` | **0** (DELETE'lendi) |
| 2 | `SELECT COUNT(*) FROM products WHERE specs ? 'substrate_safe'` | **0** (merge) |
| 3 | `SELECT COUNT(*) FROM products WHERE specs ? 'safe_on'` | **9** |
| 4 | `SELECT COUNT(*) FROM products WHERE specs ? 'equipment_compat'` | **3** |
| 5 | `SELECT specs->>'safe_on' FROM products WHERE sku IN ('700507','71490','79290','Q2M-EW1000M','Q2M-RWYA1000M')` | hepsi `["ceramic_coating"]` |
| 6 | `SELECT specs->>'safe_on' FROM products WHERE sku IN ('PPF-L500M','Q2M-PPFSL4000M')` | hepsi `["ppf","vinil"]` |
| 7 | `SELECT specs->>'safe_on' FROM products WHERE sku='701851'` | `["aluminum","fiberglass","plexiglass"]` |
| 8 | `SELECT specs->>'safe_on' FROM products WHERE sku='75021'` | `["aluminum"]` |
| 9 | `SELECT specs->>'equipment_compat' FROM products WHERE sku='SGGD135'` | `Karcher K Serisi` |
| 10 | `SELECT COUNT(*) FROM product_meta WHERE key='compatibility'` | **0** |
| 11 | `SELECT COUNT(*) FROM product_meta WHERE key='substrate_safe'` | **0** |
| 12 | `SELECT COUNT(*) FROM product_meta WHERE key='safe_on'` | **9** |
| 13 | `SELECT COUNT(*) FROM product_meta WHERE key='equipment_compat'` | **3** |
| 14 | search_text snippet'inde `Üzerinde güvenli:` geçen ürün | **9** |
| 15 | search_text snippet'inde `Ekipman uyumu:` geçen ürün | **3** |
| 16 | `cd retrieval-service && bun tsc --noEmit` | bizim diff ile yeni error yok |
| 17 | `adk build` | OK |

Commit: `fix(bot+db): Phase 1.1.13D — compatibility split (safe_on + equipment_compat) + substrate_safe merge`

## Critical files

- **Yeni:** [retrieval-service/scripts/phase-1-1-13d-compat-split.ts](retrieval-service/scripts/phase-1-1-13d-compat-split.ts) — migration (audit + --apply + guards)
- **Edit:** [retrieval-service/scripts/project-specs-to-meta.ts](retrieval-service/scripts/project-specs-to-meta.ts) — SCALAR/ARRAY/STALE keys
- **Edit:** [retrieval-service/scripts/regenerate-search-text.ts](retrieval-service/scripts/regenerate-search-text.ts) — explicit safe_on + equipment_compat snippet
- **Edit:** [Botpress/detailagent-ms/src/tools/search-products.ts](Botpress/detailagent-ms/src/tools/search-products.ts) — 5 yer (description + örnekler + ARRAY listesi + SCALAR listesi + uyarı)
- **Edit:** [Botpress/detailagent-ms/src/conversations/index.ts](Botpress/detailagent-ms/src/conversations/index.ts) — 5 satır (379, 623, 624, 633, 643)
- **Edit:** [Botpress/detailagent-ms/src/tools/get-product-details.ts](Botpress/detailagent-ms/src/tools/get-product-details.ts) — 50-51 satır (safe_on + equipment_compat description)

## Out of Scope (sonraki plan'lara taşındı)

- **safe_on enrichment** — kullanıcı kararı: 484 üründe meta key olması saçma (mekanik aksesuar). Sadece kimyasal ürünlerde anlamlı, gerekirse sonraki bir fazda kategori bazında enrichment.
- **Phase 1.1.10b** — clarifying-first refactor
- **ph_tolerance normalize** — kaplama ürünlerinde "2-11" range string
- **434 ürün için pH enrichment** — kullanıcı kararı: önemli değil, skip

## Süre tahmini

~35 dk (migration ~15 + projector ~5 + search-text/embed ~5 + verify+commit ~10).
