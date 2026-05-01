# Menzerna Marka Sayfası - UI/UX & Frontend İnceleme ve İyileştirme Planı

## Context

Menzerna marka sayfası plugin'i (`mgps-brand-menzerna`) tamamlanmış Gyeon referans plugin'i temel alınarak oluşturulmuş. Menzerna'nın endüstriyel kimliği (kırmızı-siyah-beyaz, keskin köşeler, Source Sans 3) doğru yansıtılmış, ancak preview.html ile shortcode PHP dosyaları arasında önemli uyumsuzluklar ve çeşitli UI/UX iyileştirme fırsatları tespit edildi.

Plugin yapısı: Hero → Story → Features → 9 Kategori (her birine ürün carousel ekleniyor)

**Kapsam:** Tam kapsamlı (5 kritik + 11 önemli + 9 polish) + tüm kategorilere ürün yerleşimi

---

## Kritik Sorunlar (Lansmandan Önce Düzeltilmeli)

### K1. Font Yükleme — Inline `<link>` yerine `wp_enqueue_scripts`
**Dosya:** `mgps-brand-menzerna.php`

Source Sans 3 fontu `shortcode-hero.php` içinde `<body>` tag'ı ortasında `<link>` ile yükleniyor. Bu:
- Geçersiz HTML (FOUT riski)
- Hero shortcode kullanılmazsa font hiç yüklenmez
- Gyeon bunu `wp_enqueue_scripts` hook'u ile `<head>` içinde çözüyor

**Çözüm:** Font yüklemeyi `mgps-brand-menzerna.php` ana dosyasına taşı, `wp_enqueue_scripts` ve `wp_head` action'ı kullan.

### K2. Scroll Reveal Animasyonları Shortcode'larda Yok
**Dosyalar:** Tüm `includes/shortcode-*.php` dosyaları

Preview HTML'de `.mz-reveal` class'ı ve IntersectionObserver JS mevcut, ancak **hiçbir shortcode PHP dosyası** bu class'ı veya JS'i çıktıya eklemiyor. Canlı WordPress sitesinde **hiçbir section scroll'da animasyonlu girmeyecek**.

**Çözüm:** Her shortcode çıktısına `.mz-reveal` class'ı ekle ve IntersectionObserver JS'ini bir kez yükle (Gyeon'un `window.__mzRevealInit` kontrol paterni ile).

### K3. `prefers-reduced-motion` Desteği Eksik
**Dosyalar:** Tüm CSS çıktıları

Hareket hassasiyeti olan kullanıcılar için hiçbir destek yok.

**Çözüm:**
```css
@media(prefers-reduced-motion:reduce){
    .mz-hero__content{animation:none;opacity:1;transform:none}
    .mz-hero__scroll-line{animation:none}
    .mz-reveal{opacity:1;transform:none;transition:none}
    .mz-feat,.mz-prod{transition:none}
}
```

### K4. Erişilebilirlik Eksikleri
**Dosyalar:** `shortcode-hero.php`, `shortcode-products.php`

- YouTube iframe'de `title` attribute eksik
- Carousel dot butonlarında `aria-label` eksik
- Dot butonlarında `:focus-visible` style yok
- SVG ikonlarında `aria-hidden="true"` eksik
- Carousel section'larında `role="region"` ve `aria-label` eksik

### K5. YouTube iframe Performans Sorunu
**Dosya:** `shortcode-hero.php`

YouTube iframe sayfa yüklenirken hemen yükleniyor (2-5MB ek yük). LCP ve FCP'yi ciddi etkiler.

**Çözüm:** Poster image göster, tıklanınca iframe yükle (lite-youtube-embed paterni) veya self-hosted MP4 kullan.

---

## Önemli İyileştirmeler (Kalite Farkı Yaratan)

### Ö1. Carousel `scroll-padding-left` Uyumsuzluğu
**Dosya:** `shortcode-products.php`

Carousel: `scroll-padding-left: 30px`, Container: `padding: 0 150px`. Snap edilen kartlar sol kenarla hizalanmıyor.

**Çözüm:** `scroll-padding-left: 150px` yap (960px'de `24px`, 480px'de `16px`).

### Ö2. Carousel Kart Genişliği Sınırsız
Ultrawide ekranlarda (2560px+) kartlar aşırı geniş (~515px). Gyeon `min-width` kullanıyor.

**Çözüm:** `.mz-prod` veya `.mz-products__inner` için `max-width` ekle.

### Ö3. Features Grid 2 Sütun Ara Durum Eksik
960px'de direkt 3 sütundan 1 sütuna düşüyor. Tablet'te 2 sütun ara durum daha iyi.

**Çözüm:** 960px'de `grid-template-columns: repeat(2, 1fr)`, 480px'de `1fr`.

### Ö4. Carousel Padding Sert Geçiş
Desktop'ta 150px, 960px'de ani 24px'e düşüyor (126px fark).

**Çözüm:** Ara breakpoint ekle: 1200px'de 80px, 960px'de 40px, 480px'de 16px.

### Ö5. Tüm Dark Kategoriler Aynı Gradient
Gyeon her kategori için benzersiz ton kullanıyor. Menzerna'nın 5 dark kategorisi tamamen aynı `#1a1a1a → #0a0a0a`.

**Çözüm:** Her kategorinin accent rengine çok hafif tint ekle:
- Heavy Cut: `#1a1818 → #0a0909` (kırmızımsı)
- Finishing: `#181a18 → #090a09` (yeşilimsi)
- Marine: `#18191a → #090a0a` (mavimsi)

### Ö6. Inaktif Dot Kontrast Sorunu (WCAG)
`rgba(255,255,255,.15)` / `#1a1a1a` arka plan = ~2.1:1 oran. WCAG AA 3:1 gerektirir.

**Çözüm:** Opacity'yi `.25`'e yükselt.

### Ö7. Font-Family Tekrarı
Her element'e ayrı `font-family` yazılmış. Container seviyesinde tek deklarasyon yeterli.

### Ö8. Letter-Spacing Tutarsızlığı
Label'lar `.3em`, `.35em`, `.4em` arası değişiyor. 2-3 tier'e sadeleştir.

### Ö9. `loading="lazy"` Eksik
Story ve kategori görsellerinde lazy loading yok. Gyeon bunu kullanıyor.

### Ö10. 768-960px Arası Uzun Metin Satırları
Kategori section'larında 80+ karakter satır uzunluğu. `max-width: 640px` veya daha fazla padding gerekli.

### Ö11. CSS Tekrarı — Repeated Shortcode Çıktıları
`[mgps_menzerna_category]` 9 kez kullanılınca CSS 9 kez emit ediliyor (~18KB gereksiz).

**Çözüm:** CSS'i flag kontrolü ile bir kez yaz veya `wp_enqueue_style` ile inline style kullan.

---

## İyileştirmeler (Polish)

### P1. Ürün Kartı Görsel Hover Eksik
Gyeon'da `scale(1.05)` var, Menzerna'da yok.

### P2. Feature Kart Staggered Animasyon
Gyeon'da kademeli giriş animasyonu var (120ms gecikme).

### P3. Body Text line-height Tutarsızlığı
Story: 1.85, Features: 1.75, Category: 1.8 — Tek değer (1.8) kullan.

### P4. Hero Scroll Indicator İnteraktif Değil
`<a href="#story">` ile tıklanabilir yapılmalı.

### P5. Carousel Overflow Yokken Dot Gizleme
4 ürün tam sığınca scroll yok ama dot'lar görünüyor.

### P6. Cubic-bezier Easing
`ease` yerine `cubic-bezier(.25,.46,.45,.94)` daha profesyonel.

### P7. Kategori Görseli Arkasında Glow Efekti
Gyeon'da `::before` ile `blur(60px)` radial glow var.

### P8. Weight 900 Sadece Watermark İçin
3% opacity'de 700 ile 900 farkı görülmez, 900 kaldırılabilir (~15KB tasarruf).

---

## Menzerna'nın İyi Yaptıkları (Korunacak)

1. **Kategori accent renk sistemi** — Her seriye özel renk (kırmızı, gold, yeşil, mavi, cyan, gri) Gyeon'dan daha zengin
2. **Counter/stats section** — 4'lü istatistik satırı marka güvenilirliğini artırıyor
3. **Kontrollü max-width (1280px)** — Gyeon'un sadece padding yaklaşımından daha tutarlı
4. **3 sütun features grid** — Daha okunaklı kart boyutu
5. **Keskin köşeler** — Menzerna endüstriyel kimliğine sadık
6. **Solid CTA (hero) vs Outline CTA (kategori)** — Doğru hiyerarşi

---

## Uygulama Sırası

### Faz 0: Ürün Yerleşimi (Öncelik)
0a. Mevcut carousel'lerin (Heavy, Medium, Finishing, Protection) ürün URL'lerini gerçek slug'larla güncelle
0b. Yeni carousel section'ları ekle: Marine, Industrial, Foam Pads, Wool, Tools
0c. Carousel JS'e "overflow yoksa dot gizle" mantığı ekle
0d. preview.html'i tüm kategoriler + ürünlerle güncelle

### Faz 1: Kritik Düzeltmeler
1. Font yüklemeyi `mgps-brand-menzerna.php`'ye taşı (K1)
2. Scroll reveal'ı tüm shortcode'lara ekle (K2)
3. `prefers-reduced-motion` desteği ekle (K3)
4. Erişilebilirlik düzeltmeleri (K4)
5. YouTube lazy-load implementasyonu (K5)

### Faz 2: Carousel & Responsive İyileştirmeler
6. `scroll-padding-left` düzelt (Ö1)
7. Carousel max-width ekle (Ö2)
8. Features 2-sütun tablet state (Ö3)
9. Carousel padding kademeli geçiş (Ö4)
10. İnaktif dot kontrast düzelt (Ö6)
11. CSS tekrarını önle (Ö11)

### Faz 3: Görsel Tutarlılık
12. Dark gradient tint'ler (Ö5)
13. Font-family sadeleştirme (Ö7)
14. Letter-spacing harmonizasyonu (Ö8)
15. `loading="lazy"` ekle (Ö9)
16. line-height birleştirme (P3)

### Faz 4: Polish & Enhancement
17. Ürün hover scale (P1)
18. Feature stagger animasyon (P2)
19. Scroll indicator interaktif (P4)
20. Cubic-bezier easing (P6)
21. Kategori glow efekti (P7)

---

## Ürün Yerleşimi — Tüm Kategoriler

Kural: >4 ürün → scroll + dot navigator aktif, ≤4 ürün → statik grid, dot gizli.

### 1. Heavy Cut Series (Dark, Left) — 7 ürün ✓ Carousel
| # | Ürün | URL Slug |
|---|------|----------|
| 1 | Menzerna AS 30 Ağır Çizik Giderici | `menzerna-as-30-agir-cizik-giderici-krem-kalin-pasta-1-kg` |
| 2 | Menzerna Likit Matlaştırıcı Pasta | `menzerna-likit-matlastirici-pasta-1-lt` |
| 3 | Menzerna 400 Yeşil Seri Kalin Pasta | `menzerna-400-yesil-seri-agir-cizik-giderici-kalin-pasta-1-lt` |
| 4 | Menzerna Yeni 400 Kalın Pasta | `menzerna-yeni-400-agir-cizik-giderici-kalin-pasta-1-kg` |
| 5 | Menzerna Cut Force Pro Altın Seri | `menzerna-cut-force-pro-altin-seri-agir-cizik-giderici-kalin-pasta-1-lt` |
| 6 | Menzerna 1100 Keçe Uyumlu Pasta | `menzerna-1100-kece-uyumlu-kalin-pasta-agresif-cizik-giderici-1-lt` |
| 7 | Menzerna 1000 Çizik Giderici | `menzerna-1000-cizik-giderici-kalin-pasta-1-kg` |

### 2. Medium Cut Series (Light, Right) — 3 ürün, statik grid
| # | Ürün | URL Slug |
|---|------|----------|
| 1 | Menzerna One-Step Polish 3in1 | `menzerna-one-step-polish-3in1-tek-adim-pasta-cila-koruma-1-lt` |
| 2 | Menzerna 2200 İnce Çizik Giderici | `menzerna-2200-ince-cizik-giderici-pasta-1-lt` |
| 3 | Menzerna 2500 İnce Çizik Giderici | `menzerna-2500-ince-cizik-giderici-pasta-1-lt` |

### 3. Finishing Series (Dark, Left) — 3 ürün, statik grid
| # | Ürün | URL Slug |
|---|------|----------|
| 1 | Menzerna 3000 Standart Hare Giderici | `menzerna-3000-standart-hare-giderici-cila-1-lt` |
| 2 | Menzerna 3500 Premium Hare Giderici | `menzerna-3500-premium-hare-giderici-cila-1-lt` |
| 3 | Menzerna 3800 Süper Hare Giderici | `menzerna-3800-super-hare-giderici-cila-1-lt` |

### 4. Paint Protection Series (Light, Right) — 3 ürün, statik grid
| # | Ürün | URL Slug |
|---|------|----------|
| 1 | Menzerna Sealing Wax Boya Koruma | `menzerna-sealing-wax-boya-koruma-cilasi-1-lt` |
| 2 | Menzerna Power Lock Üstün Koruma | `menzerna-power-lock-ustun-boya-koruma-cilawax-1-lt` |
| 3 | Menzerna Ceramic Spray Sealant | `menzerna-ceramic-spray-sealant-seramik-icerikli-sprey-boya-koruyucu-cila-500-ml` |

### 5. Marine Polishing Series (Dark, Left) — 2 ürün, statik grid
| # | Ürün | URL Slug |
|---|------|----------|
| 1 | Menzerna Marin 3in1 Tekne Gelcoat | `menzerna-marin-tekne-gelcoat-boya-3in1-tek-adim-pasta-cila-koruma-1-lt` |
| 2 | Menzerna Premium Power Cut 200 | *Harici: menzerna.com/boat-care/boat-polish — yerel ürün yoksa link verilecek* |

### 6. Industrial Polishing (Light, Right) — 11 ürün ✓ Carousel
| # | Ürün | URL Slug |
|---|------|----------|
| 1 | Menzerna 113GZ Kesici Katı Cila | `menzerna-113gz-kesici-kati-cila-kahverengi-pirinc-zamak-aluminyum-parlatici-1100-gr` |
| 2 | Menzerna 439T Kesici Katı Cila | `menzerna-439t-kesici-kati-cila-yesil-paslanmaz-celik-aluminyum-pirinc-parlatici-1100-gr` |
| 3 | Menzerna 495P Parlatıcı Katı Cila | `menzerna-495p-parlatici-kati-cila-beyaz-plastik-kompozit-degerli-metaller-boyali-yuzey-parlatici-1250-gr` |
| 4 | Menzerna 480W Parlatıcı Katı Cila | `menzerna-480w-parlatici-kati-cila-bej-pirinc-krom-zamak-aluminyum-kompozit-parlatici-1200-gr` |
| 5 | Menzerna P14F Orta Kesici Katı Cila | `menzerna-p14f-orta-kesici-kati-cila-beyaz-aluminyum-pirinc-paslanmaz-tek-adim-parlatici-1300-gr` |
| 6 | Menzerna P126 Parlatıcı Katı Cila | `menzerna-p126-parlatici-kati-cila-pembe-paslanmaz-celik-degerli-metal-parlatici-1300-gr` |
| 7 | Menzerna M5 Süper Parlatıcı | `menzerna-m5-super-parlatici-kati-cila-beyaz-degerli-metaller-paslanmaz-celik-boyali-yuzey-parlatici-1300-gr` |
| 8 | Menzerna P164 Orta Kesici Katı Cila | `menzerna-p164-orta-kesici-kati-cila-mavi-cok-amacli-yuzey-parlatici-1250-gr` |
| 9 | Menzerna P175 Süper Parlatıcı | `menzerna-p175-super-parlatici-kati-cila-sari-cok-amacli-yuzey-parlatici-1300-gr` |
| 10 | Menzerna GW18 Kesici Katı Cila | `menzerna-gw18-kesici-kati-cila-bej-plastik-kompozit-boyali-yuzey-parlatici-1150-gr` |
| 11 | Menzerna GW16 Parlatıcı Katı Cila | `menzerna-gw16-parlatici-kati-cila-bej-plastik-kompozit-boyali-yuzey-parlatici-1200-gr` |

### 7. Foam Polishing Pads (Dark, Left) — 7 ürün ✓ Carousel
| # | Ürün | URL Slug |
|---|------|----------|
| 1 | Menzerna P150M Hare Giderici Süngeri Sarı 150mm | `menzerna-p150m-hare-giderici-cila-sungeri-sari-150mm` |
| 2 | Menzerna Ağır Kesim Pasta Süngeri Kırmızı 150mm | `menzerna-agir-kesim-pasta-sungeri-kirmizi-150mm` |
| 3 | Menzerna İnce Kesim Pasta Süngeri Sarı 150mm | `menzerna-ince-kesim-pasta-sungeri-sari-150mm` |
| 4 | Menzerna Hare Giderici Cila Süngeri Yeşil 150mm | `menzerna-hare-giderici-cila-sungeri-yesil-150mm` |
| 5 | Menzerna Ağır Kesim Pasta Süngeri 2'li Kırmızı 95mm | `menzerna-agir-kesim-kalin-pasta-sungeri-2li-paket-kirmizi-95mm` |
| 6 | Menzerna İnce Kesim Pasta Süngeri 2'li Sarı 95mm | `menzerna-ince-kesim-ara-kat-pasta-sungeri-2li-paket-sari-95mm` |
| 7 | Menzerna Hare Giderici Cila Süngeri 2'li Yeşil 95mm | `menzerna-hare-hologram-giderici-cila-sungeri-2li-paket-yesil-95mm` |

### 8. Wool Buffing Pads (Light, Right) — 4 ürün, statik grid (dot gizli)
| # | Ürün | URL Slug |
|---|------|----------|
| 1 | Menzerna Krater Kuzu Yünü Keçe 220mm | `menzerna-krater-kuzu-yunu-polisaj-kece-220mm9` |
| 2 | Menzerna Premium Orbital Kuzu Yünü 165mm | `menzerna-premium-orbital-kuzu-yunu-polisaj-kecesi-165mm6-5` |
| 3 | Menzerna Premium Orbital Kuzu Yünü 140mm | `menzerna-premium-orbital-kuzu-yunu-polisaj-kecesi-140-mm5-5` |
| 4 | Menzerna Premium Kuzu Yünü 90mm | `menzerna-premium-kuzu-yunu-polisaj-kecesi-90-mm3-5` |

### 9. Polishing Tools & Accessories (Dark, Left) — 3 ürün, statik grid
| # | Ürün | URL Slug |
|---|------|----------|
| 1 | Menzerna Rotary Ped Destek Disk 148mm | `menzerna-rotary-polisaj-makineleri-icin-esnek-ve-dayanikli-ped-destek-disk-tabanlik-148mm` |
| 2 | Menzerna Polisaj Dayanıklı Tabanlığı 123mm | `menzerna-polisaj-dayanikli-tabanligi-kirmizi-123mm` |
| 3 | Menzerna Rotary/Matkap Ped Destek Disk 75mm | `menzerna-rotary-polisaj-makineleri-matkap-icin-dayanikli-ped-destek-disk-tabanlik-75mm` |

### Carousel Özet
| Kategori | Ürün Sayısı | Carousel | Tema |
|----------|-------------|----------|------|
| Heavy Cut | 7 | ✓ Scroll aktif | Dark |
| Medium Cut | 3 | Statik grid | Light |
| Finishing | 3 | Statik grid | Dark |
| Protection | 3 | Statik grid | Light |
| Marine | 2 | Statik grid | Dark |
| Industrial | 11 | ✓ Scroll aktif | Light |
| Foam Pads | 7 | ✓ Scroll aktif | Dark |
| Wool Pads | 4 | Statik (4 kart sığıyor) | Light |
| Tools | 3 | Statik grid | Dark |

---

## Kritik Dosyalar

| Dosya | Değişiklik |
|-------|-----------|
| `mgps-brand-menzerna.php` | Font yükleme, global CSS, CSS dedup flag |
| `includes/shortcode-hero.php` | YouTube lazy-load, a11y, font link kaldır |
| `includes/shortcode-story.php` | Scroll reveal, lazy loading, a11y |
| `includes/shortcode-features.php` | Scroll reveal, stagger, 2-col tablet |
| `includes/shortcode-category.php` | Scroll reveal, gradient tint, a11y |
| `includes/shortcode-products.php` | Carousel fixes, a11y, dot contrast |
| `preview.html` | Tüm CSS/JS değişikliklerini yansıt |

---

## Doğrulama

1. **Preview HTML** tarayıcıda aç — tüm breakpoint'lerde kontrol et (320, 480, 768, 960, 1280, 1920, 2560px)
2. **WordPress** shortcode'ları test sayfasında render et
3. **Lighthouse** accessibility ve performance audit çalıştır
4. **Klavye navigasyonu** test et (Tab ile carousel dot'larına eriş)
5. **prefers-reduced-motion** OS ayarını aktif edip test et
6. **Preview ↔ Plugin CSS** birebir eşitlik kontrolü
