# `admin-ui-design-plan.md` — Amaç ve İçerik Özeti

## Context

Kullanıcı, [docs/design/admin-ui-design-plan.md](docs/design/admin-ui-design-plan.md) dosyasının (2436 satır, 16 bölüm + 2 ek) **tam amacını** netleştirmek istiyor. Bu plan dosyası **kod değil, tasarım şartnamesi** — ayrı bir oturumda implement edilmek üzere yazılmış self-contained bir spec. Bu plan dosyası, asıl dokümanın "ne için var" sorusuna net cevap üretmek içindir.

---

## 1. Dokümanın amacı — tek cümle

> **511 ürünlük MTS Kimya detailing kataloğunun veri kalite sorunlarını operatör hızında çözmeye izin verecek, sıcak-tonlu yerel bir admin web arayüzünün (Catalog Atelier) tasarım şartnamesi.**

---

## 2. Hangi problemi çözüyor?

Asıl doküman §1'de şu somut veri kalite sorunlarını sıralıyor:

| Problem | Örnek | Sonucu |
|---|---|---|
| **Fragmentation** | 511 ürün, 26 template_group, 156 sub_type — parçalanmış | Katalog yönetilemez durumda |
| **Duplicate keys** | `ph`, `ph_level`, `ph_tolerance` üç ayrı key | Bot cevap üretirken hangisine baksın bilemiyor |
| **Sibling sprawl** | `durability_months`, `durability_km`, `durability_years`, ... 9 kardeş alan | Normalize edilmemiş |
| **Null hotspot** | `silicone_free` sadece seramiklerin %4'ünde dolu | Bot silikon sorusuna **uyduruyor** |
| **Type inconsistency** | `cut_level` kimi yerde string "3", kimi yerde number 3 | Arama/filtre bozuluyor |
| **Missing FAQ** | 60 ürünün hiç FAQ'i yok | Bot template cevap üretemiyor |
| **Fiyat = 0** | 1 ürün (Menzerna PPC 200) | Kasıtsız listeleme |

**Kritik satır (§1):** *"Bu arayüz, uydurmanın alternatifini operatöre teslim etmek için var."*

---

## 3. Çözüm yaklaşımı — tasarım felsefesi

Doküman 4 temel ilke koyuyor:

### 3.1 Heatmap-first navigation
Landing page **dashboard değil**, `template_group × specs_key` coverage ısı haritası. Eksik veri noktaları (örn. ceramic × silicone_free = %4) ekran açılışında kırmızı gözükür, tıklama → ilgili ürünler.

### 3.2 Staging-first write flow
Hiçbir edit direkt DB'ye yazmaz. Her değişiklik **Zustand + localStorage** staged store'a düşer; sağ kenardaki Staging Drawer'da her zaman görünür; `Commit` butonu tek SQL transaction'da uygular.
*Neden:* rollback + diff görünürlüğü + multi-operator güvenlik.

### 3.3 Keyboard-first operator speed
- `⌘K` command palette — sadece nav değil, **data transformation** komutları ("Normalize numeric fields", "Merge ph/ph_level/ph_tolerance → ph")
- `⌘J` staging drawer toggle
- `⌘Z` / `⌘⇧Z` undo / redo
- `⌘⇧↵` commit workflow

### 3.4 Warm editorial aesthetic — "Warm Archive Atelier"
Bilinçli olarak purple/indigo SaaS tuzağından kaçış:
- **Renk:** cream `#FAF6ED` zemin + terracotta `#C65D3F` accent + sage `#7A8B56` secondary + amber staging glow
- **Tipografi:** Fraunces serif (başlık) + IBM Plex Sans (body) + IBM Plex Mono (veri)
- **Doku:** paper grain (SVG noise %4 opacity)
- **Hissi:** editoryal dergi + zanaatkar tezgâhı, **laboratuvar sterilliği değil**

---

## 4. Dokümanın yapısı — 16 bölüm + 2 ek

| # | Bölüm | Amaç |
|---|---|---|
| §1 | Executive Summary | Problem + çözüm bir sayfada |
| §2 | Tasarım felsefesi | Aesthetic direction ve "ne DEĞİL" listesi |
| §3 | Design tokens — color | OKLCH + HEX, primitive → semantic map, WCAG kontrast tablosu |
| §4 | Typography scale | Font stack, modular scale (1.250), mood örnekleri |
| §5 | Bilgi mimarisi | 20+ route ağacı, top bar + left rail, persistent surfaces |
| §6 | Wireframes (10 ekran) | Dashboard, Catalog Tree, Product Editor (6 tab), Bulk, Heatmap, FAQ, Relations, Staging, Commit, Command Palette — ASCII sketch + component listesi |
| §7 | Etkileşim akışları | 6 kritik user flow (inline edit, bulk, taxonomy, FAQ generate, specs normalize, keyboard escapes) |
| §8 | Component library (17 component) | SpecEditor, VariantEditor, RelationGraph, Heatmap, DiffViewer, CommandPalette, StagingDrawer, TaxonomyTree, InlineEditCell, StatusBadge, **PromptSectionEditor**, **ToolRegistryEditor**, **TokenMeter**, **DiffPromptViewer**, **PromptPlayground**, **AgentSwitcher**, shared primitives |
| §9 | Tech stack | Next.js 15 + Tailwind v4 + shadcn + TanStack Query/Table + Zustand + dnd-kit + react-flow + cmdk + Monaco — justification ile |
| §10 | Admin API layer (14 grup) | `retrieval-service`'e eklenecek `/admin/*` endpoint'leri (products, faq, relations, meta, taxonomy, coverage, staging, export, history, embedding, agents, prompts, tools, version history) |
| §11 | Data flow | Read (Supabase RLS read-only) vs Write (staging → admin API → tek transaction commit) |
| §12 | Edge case'ler | Concurrent edit, validation, undo, network failure, bulk chunk, embedding backlog, migration lock, LLM halüsinasyon, soft delete, fiyat=0 |
| §13 | Accessibility | WCAG 2.2 AA hedefi, keyboard nav, screen reader, reduced motion, heatmap a11y |
| §14 | Implementation phases | **Phase 0 scaffolding → Phase 5 polish**, toplam 6-8 hafta |
| §15 | Test senaryoları | 10 senaryo kategorisi (render, product edit, bulk, FAQ gen, normalize, taxonomy, a11y, performance, export, dark) |
| **§16** | **Prompt Lab — derin dalış** | Bot instruction editor + tool registry + token meter + playground + version history — ayrı bir sub-module |
| Ek A | Command palette komut listesi | Tüm `⌘K` komutları kategorize |
| Ek B | CSS variable referansı | Tailwind v4 `@theme` directive özeti |

---

## 5. Prompt Lab — dokümanın gizli ikinci amacı

§16 (bölümün %20'si) salt veri editleme için değil, **bot'un kendisini düzenlemek için**:

- `Botpress/detailagent-ms/src/conversations/index.ts` — 6536 token'lık instruction'ı **section bazlı** düzenlemek (hedef 6000)
- `Botpress/detailagent-ms/src/tools/*.ts` — 7 tool'un description + Zod schema'larını görsel editor'da yönetmek
- **Token meter** — hangi section ne kadar token yiyor, hangi section data-fix ile kaldırılabilir (SPEC-FIRST section'ı -450 tok, PROACTIVE+RELEVANCE merge -350 tok)
- **Prompt playground** — compiled prompt preview + Gemini dry-run simulation (mock veya live, rate-limit ile)
- **Version history + rollback** — git commit bazlı diff + staging üzerinden rollback

**Neden bu kısım var:** Phase 4 Round 2'de bot instruction'ı 800 token şişti (10k → 16k input), v10.1'de 14-error backtick bug'ı yaşandı. Prompt Lab, hem bloat kontrolü hem syntax guard hem de davranış simülasyonunu tek ekranda topluyor.

---

## 6. Implementation kapsamı

Dokümanın §14'üne göre (solo dev için tahmini süreler):

```
Phase 0  (1-2 gün)   Scaffolding — Next.js init, design tokens, fonts, layout shell
Phase 1  (1 hafta)   MVP — Dashboard heatmap + Catalog Tree + read-only product detail
Phase 2  (2 hafta)   V1 — Inline edit + staging drawer + commit + undo/redo
Phase 3  (1 hafta)   V1.5 — FAQ editor + Relations table + history + CSV/JSONL export
Phase 4  (2 hafta)   V2 — Bulk ops + normalization + taxonomy + relations graph +
                         **Prompt Lab** (8 alt-faz) + LLM FAQ generation
Phase 5  (1 hafta)   V2.5 — A11y + performance + error boundaries + print styles
                     ──────────
                     Toplam: 6-8 hafta (solo) / 3-4 hafta (2 paralel dev)
```

---

## 7. Kritik dosya referansları

Plan uygulandığında etkilenecek / oluşacak dosyalar:

| Yol | Durum | İçerik |
|---|---|---|
| [docs/design/admin-ui-design-plan.md](docs/design/admin-ui-design-plan.md) | **var** (2436 satır) | Bu referans doküman |
| [retrieval-service/src/routes/](retrieval-service/src/routes/) | **var** | Admin API buraya eklenecek (§10) |
| [Botpress/detailagent-ms/src/conversations/index.ts](Botpress/detailagent-ms/src/conversations/index.ts) | **var** | Prompt Lab'ın hedef dosyası |
| [Botpress/detailagent-ms/src/tools/](Botpress/detailagent-ms/src/tools/) | **var** | 7 tool — Tool Registry Editor'ın hedefi |
| `admin-ui/` | **yok** | Next.js 15 projesi — Phase 0'da oluşturulacak |

---

## 8. Verification — "amacı anladım" kontrol listesi

Plan'ın amacını doğru anlamak için şu soruların cevaplarını doğrulamak gerek:

1. ✅ **Kim kullanacak?** → MTS Kimya'nın internal veri operatörleri (teknik bilgili ama dev değil)
2. ✅ **Nerede çalışacak?** → Yerel / internal web arayüzü (production public değil)
3. ✅ **Veriye nasıl yazacak?** → Direkt DB değil, staging → batch commit (rollback-safe)
4. ✅ **Neyi değiştirmeyecek?** → Bot runtime, Supabase schema (migration önceden yapılacak), eval suite
5. ✅ **Neyi çözecek?** → Null hotspot'lar, duplicate keys, type inconsistency, missing FAQs, bot instruction bloat
6. ✅ **Hangi estetikle?** → Warm terracotta/sage/cream editorial (SaaS purple değil)
7. ✅ **Ne kadar sürer?** → 6-8 hafta (solo), faz-faz ilerletilebilir

---

## 9. Sonuç — dokümanın tek cümleyle yeniden özeti

**Bu doküman, kod yazmak değil, bir sonraki oturumda başka bir AI ya da developer kod yazarken "her karar için tek bir kaynağa bakabileceği" bir implementation contract'tır.** Self-contained — design tokens, wireframes, component API'leri, endpoint specs, edge case handling, test senaryoları, implementation phases ve ek olarak Prompt Lab sub-modülü hepsi tek dosyada toplanmış. Amacı; veri kalite sorunlarının operatör refleksine çevrilebilmesi ve bu çeviriyi yapan arayüzün tasarımının belirsiz kalmaması.
