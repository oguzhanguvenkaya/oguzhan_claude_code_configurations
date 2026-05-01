# Plan: Google Labs `DESIGN.md` Repo İncelemesi ve MG Polishing'e Uygunluk Analizi

## Context

Kullanıcı, [google-labs-code/design.md](https://github.com/google-labs-code/design.md) repo'sunu ve [Stitch + DESIGN.md tanıtım blog yazısını](https://blog.google/innovation-and-ai/models-and-research/google-labs/stitch-design-md/) inceleme istedi. MG Polishing projesinde halihazırda `claude-design/` altında benzer amaçlı dosyalar mevcut: `flex-brand-guide.md` (374 satır), `flex-design-tokens.json` (4.4 KB), `flex-section-templates.md`. IDE'de `flex-brand-guide.md` açık. Amaç: DESIGN.md spec'ini anlamak ve mevcut marka rehberi yaklaşımıyla karşılaştırarak benimsenip benimsenmemesi gerektiğine karar vermek için bir analiz raporu üretmek.

Bu plan **kod değişikliği değil**, salt **rapor/öneri** üretir. Uygulama (migrasyon) için ayrı bir plan açılması önerilir.

---

## 1. DESIGN.md Nedir? (Özet)

**Tek cümle**: AI coding agent'larının bir markanın görsel kimliğini tutarlı şekilde uygulayabilmesi için, makine-okunabilir YAML token'ları + insan-okunabilir Markdown gerekçeleri bir araya getiren açık spesifikasyon.

**Yapı**:
- YAML frontmatter (`---` ile) → token'lar (colors, typography, spacing, components)
- Markdown gövde → kanonik sıra: Overview → Colors → Typography → Layout → Elevation & Depth → Shapes → Components → Do's and Don'ts
- Token referans syntax'ı: `{colors.primary}`

**CLI** (`@google/design.md`):
- `lint` — 7 kural (kırık referans, eksik primary color, WCAG AA contrast, orphan token, section sıralaması…)
- `diff` — sürümler arası regresyon tespiti
- `tailwind` — Tailwind config üretimi
- `spec` — agent prompt'una eklenmek üzere format spec'i

**Durum**: alpha, draft.

**Stitch entegrasyonu**: Google Labs'ın UI tasarım aracı Stitch, DESIGN.md dosyalarını import/export edebiliyor; agent "rengin ne için kullanıldığını biliyor, tahmin etmiyor".

---

## 2. MG Polishing Mevcut Yaklaşımıyla Karşılaştırma

| Boyut | DESIGN.md (Google) | MG Polishing (mevcut) |
|---|---|---|
| Dosya yapısı | Tek dosya: YAML+MD | İkili: `*-brand-guide.md` + `*-design-tokens.json` (+ `*-section-templates.md`) |
| Token formatı | YAML frontmatter | Ayrı JSON dosyası |
| Token referans | `{colors.primary}` syntax'ı | Manuel hex tekrarı, JSON key'leri |
| Section sırası | Spec'le sabit kanonik sıra | Esnek, marka bazlı (14 section'a kadar) |
| Tipografi tanımı | fontFamily/Size/Weight/lineHeight/letterSpacing | Type scale 11 rol + transform kuralları |
| Erişilebilirlik | Lint: WCAG AA contrast otomatik | Manuel: "WCAG AA verified" notu |
| Do's/Don'ts | Spec'te zorunlu section | Mevcut (Anti-patterns) |
| Otomatik araç | CLI: lint, diff, tailwind | Yok (manuel preview.html) |
| Tailwind/CSS üretimi | `tailwind` komutu | Vanilla CSS, Tailwind kullanılmıyor |
| Kapsam | Görsel kimlik + bileşen token'ları | + section templates + prompt örnekleri + WP shortcode haritası |

**Kritik farklar**:
- MG Polishing rehberi DESIGN.md'den **daha kapsamlı** (section template'leri, suggested prompts, WP composition order). Bunlar DESIGN.md spec'inde yok.
- DESIGN.md, **token doğrulama otomasyonu** sunuyor — MG Polishing'de bu yok; manuel preview.html ile görsel kontrol yapılıyor.
- DESIGN.md tek dosya ile sade, MG Polishing 3 dosya ile zengin ama dağınık.

---

## 3. Uygunluk Değerlendirmesi

### Artılar (DESIGN.md benimsemenin değeri)
1. **Standart format** → Stitch, Cursor, diğer AI araçlarıyla birlikte çalışabilirlik.
2. **Lint CLI** → marka rehberi değişikliklerinde otomatik regresyon kontrolü, contrast doğrulaması.
3. **Token referans syntax'ı** (`{colors.primary}`) → hex tekrarını ortadan kaldırır, tek kaynak prensibi.
4. **Sürüm farkı (diff)** → marka rehberi evrildiğinde neyin değiştiğini agent'a anlatmak için faydalı.
5. **Tailwind çıktısı** → ileride Tailwind'e geçilirse bedava config.

### Eksiler / Riskler
1. **Alpha durumu** → spec değişebilir; production'da kullanmak erken.
2. **Vanilla CSS uyumsuzluğu** → MG Polishing Tailwind kullanmıyor; `tailwind` komutu doğrudan yarar sağlamaz.
3. **Section template'leri ve WP shortcode haritası DESIGN.md kapsamı dışında** → ayrı dosyada tutulmaya devam edilmesi gerekir, tek-dosya avantajı kısmen kaybolur.
4. **WordPress + PHP shortcode workflow'u** → DESIGN.md, agent'ların kod üretmesini hızlandırmaya odaklı; mevcut workflow zaten manuel ve `preview.html` doğrulamalı.
5. **Türkçe içerik** → DESIGN.md prose alanlarında Türkçe yazılabilir, ama spec'in örnekleri ve community araçları İngilizce odaklı.

---

## 4. Öneri Matrisi

| Senaryo | Öneri |
|---|---|
| Hemen pilot, düşük risk | **Flex'i DESIGN.md'ye dönüştür** — mevcut JSON token'ları YAML frontmatter'a, brand-guide.md prose'unu kanonik section sırasına yerleştir. `npx @google/design.md lint` çalıştır. |
| Tüm markaları taşımak | Pilot başarılı olursa Gyeon, Menzerna, Fra-Ber için tekrarla. Section templates ve WP shortcode haritası **ayrı dosyalarda** kalsın. |
| Hibrit yol (tavsiye edilen orta yol) | DESIGN.md'nin **iyi fikirlerini** mevcut yapıya ekle: (a) JSON yerine YAML frontmatter, (b) `{colors.primary}` referans syntax'ı, (c) WCAG contrast lint script'i. Tam spec'e bağlanma. |
| Hiçbir şey yapmama | Mevcut yapı çalışıyor; alpha spec'in stabilize olmasını bekle. Sadece spec'i takipte tut. |

**Tavsiye edilen**: **Hibrit yol**. Sebep: MG Polishing rehberi DESIGN.md'den fonksiyonel olarak daha zengin (section templates, WP composition); tam migrasyon kapsam kaybına yol açar. Ama YAML frontmatter + token referans syntax'ı + WCAG lint hızlı kazanım.

---

## 5. Çıktı / Verification

Bu plan kabul edilirse, **kullanıcının onayıyla** uygulama planına geçilir:

- **Sadece rapor isterseniz**: bu plan dosyası nihai çıktıdır, ek aksiyon yok.
- **Pilot (Flex DESIGN.md)** isterseniz: yeni plan açılır, `claude-design/flex.design.md` dosyası üretilir, `npx @google/design.md lint` ile doğrulanır.
- **Hibrit** isterseniz: yeni plan açılır, `flex-brand-guide.md` üst kısmına YAML frontmatter eklenir, JSON token'lar oradan refere edilir, basit bir WCAG contrast script'i `claude-design/scripts/` altına eklenir.

**Doğrulama**:
- DESIGN.md spec referansı: https://github.com/google-labs-code/design.md (alpha)
- Mevcut yapı referansı: [claude-design/flex-brand-guide.md](claude-design/flex-brand-guide.md), [claude-design/flex-design-tokens.json](claude-design/flex-design-tokens.json)
- Hibrit yol seçilirse, lint CLI'ı global kurmadan `npx @google/design.md@alpha lint` ile test edilebilir.

---

## 6. Açık Sorular (kullanıcıya)

1. **Hedef**: sadece rapor mu, pilot mu, tüm markalar mı, hibrit mi?
2. **Tailwind**: ileride Tailwind'e geçiş düşünülüyor mu? (DESIGN.md'nin tailwind komutu o zaman değer kazanır)
3. **Stitch kullanımı**: Stitch'i deneyecek misiniz? (Eğer evetse, DESIGN.md'ye tam migrasyon mantıklı)
4. **Otomasyon önceliği**: WCAG contrast doğrulamasını otomatikleştirmek ne kadar değerli? (Hibrit yolun ana getirisi bu)
