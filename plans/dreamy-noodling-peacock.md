# Plan: Proje CLAUDE.md Oluşturma

## Context
Kullanıcı yeni chatlerde (Menzerna, Innovacar vb. markalar için) bağlam kaybı yaşamamak istiyor. Proje kök dizinine bir CLAUDE.md oluşturulacak.

## Dosya
`/Users/projectx/Local Sites/mgpolishing/CLAUDE.md`

## İçerik

```markdown
# MG Polishing — Proje Rehberi

## Proje
WordPress + WooCommerce tabanlı otomotiv detailing e-ticaret sitesi.
Local WP ile geliştirme yapılıyor. Canlı site: mgpolishing.com

## Teknoloji
- WordPress + WooCommerce + Elementor Pro
- Tema: Xprocal (parent) + xprocal-child
- PHP shortcode tabanlı custom plugin'ler
- Vanilla JS (jQuery bağımlılığı yok), CSS custom properties
- Montserrat font ailesi

## Plugin Mimarisi
Her custom plugin `mgps-` prefix'i ile isimlendirilir.
Path: `app/public/wp-content/plugins/mgps-*/`

### Plugin Pattern
- Ana dosya: `mgps-{isim}.php` — constant'lar + require
- Section'lar: `includes/shortcode-{section}.php` — her biri kendi inline CSS/JS'i ile izole
- Master shortcode: `[mgps_{isim}_page]` — tüm section'ları sırasıyla render eder
- Montserrat font plugin ana dosyasından global yüklenir, section CSS'lerinde font-family yazılmaz
- preview.html: standalone tarayıcı preview, plugin CSS ile birebir eşit tutulmalı

### Mevcut Plugin'ler
| Plugin | Shortcode | Açıklama |
|--------|-----------|----------|
| mgps-brand-gyeon | [mgps_gyeon_page] | Gyeon marka sayfası (tamamlandı) |
| mgps-ppf-series | [mgps_ppf_series] | PPF ürün carousel |
| mgps-garanti-sistem | [garanti_sorgula] | Garanti sorgulama + Elementor form entegrasyonu |
| mgps-about-page | [mgps_about_hero] vb. | Hakkımızda sayfası bileşenleri |

## Marka Sayfası Şablonu
Her marka için ayrı plugin oluşturulur: `mgps-brand-{marka}`
Bölümler sırasıyla:
1. **Hero** — Full-screen video, marka logosu, slogan, pill CTA
2. **Story** — 2 kolonlu: metin (sol) + görsel (sağ), koyu arka plan
3. **Features** — 4'lü icon/görsel kart grid, açık gri arka plan (#F5F6F8)
4. **Category + Products** — Tekrarlanan çift: category banner (koyu gradient) + ürün carousel
   - Category: sol/sağ alternatif layout, büyük watermark
   - Products: flex carousel, scroll-snap, WooCommerce slug bazlı veri çekme, dash navigator (5 çizgi)

### Layout Kuralları
- Tüm section'lar `.col-full` container'ından breakout yapar: `width:100vw; left:50%; margin-left:-50vw`
- İç padding: 180px sağ/sol (hero hariç, hero full-screen)
- Carousel: padding ile carousel scroll alanı viewport genişliğinde
- Responsive: 960px → 24px padding, 480px → 16px padding
- Desktop carousel: 4 kart + sol/sağ peek, Mobil: 2 kart + peek

### Renk Yaklaşımı
Her marka kendi brand renklerini kullanır. Renkler gyeon.co gibi resmi siteden alınır.
Renk kullanımında nötr yaklaşım:
- Metin renkleri: beyaz (#fff) koyu bg'de, #313131 açık bg'de
- Paragraf: rgba(255,255,255,.55-.65) koyu bg'de, #6B7280 açık bg'de
- Label/series: soluk tonlar (rgba .45)
- Marka rengi SADECE minimal accent'lerde (hover çizgisi, ok hover vb.)

### Ürün Carousel Detayları
- WooCommerce slug'ları ile `get_page_by_path()` → title, thumbnail, permalink
- Dash navigator: 5 sabit çizgi (40px x 3px), ortada, aktif = #313131, pasif = rgba(0,0,0,.12)
- Peek-click: kısmen görünen karta tıklayınca scroll
- Arka plan: #E4E6EB

## Kod Standartları
- BEM isimlendirme: `{marka-prefix}-{block}__{element}` (gy- Gyeon için, mz- Menzerna için vb.)
- Security: esc_html(), esc_url(), esc_attr(), wp_kses_post(), $wpdb->prepare()
- IntersectionObserver ile scroll-reveal animasyonlar
- prefers-reduced-motion desteği
- Erişilebilirlik: aria-label, keyboard navigation

## Dil
- Tüm kullanıcıya görünen metin Türkçe
- Türkçe karakterler (İ, Ö, Ü, Ş, Ç, Ğ) doğru kullanılmalı
- Marka/seri isimleri orijinal kalabilir (Q², Q²M, EVO vb.)
- text-transform: uppercase ile Türkçe i→İ dönüşümü WordPress TR locale'de çalışır

## İletişim
- Türkçe iletişim kur
- Kod yazmadan önce mevcut kodu oku
- preview.html ile plugin CSS birebir eşit tutulmalı
- Değişiklik yapınca hem plugin hem preview güncelle
```
