# Innovacar Marka Sayfası — Tasarım Revizyon Planı (Güncellendi)

## Context

Innovacar marka sayfası plugin'i (`mgps-brand-innovacar`) ilk taslak olarak oluşturuldu. Orijinal site (en.innovacar.it) incelendi ve mevcut tasarımda marka kimliğiyle uyumsuz noktalar, UI sorunları ve metin hataları tespit edildi. Bu plan, plugin'i orijinal marka kimliğine yaklaştırmak ve kalite sorunlarını gidermek için yapılacak değişiklikleri kapsar.

**Ek görev:** Tüm marka sayfası tasarım sürecinde dikkat edilen noktaları, WordPress/Elementor kurallarını ve section bazlı tasarım rehberini bir referans doküman olarak oluştur.

## Değiştirilecek Dosyalar

- `app/public/wp-content/plugins/mgps-brand-innovacar/includes/shortcode-hero.php`
- `app/public/wp-content/plugins/mgps-brand-innovacar/includes/shortcode-story.php`
- `app/public/wp-content/plugins/mgps-brand-innovacar/includes/shortcode-features.php`
- `app/public/wp-content/plugins/mgps-brand-innovacar/includes/shortcode-category.php`
- `app/public/wp-content/plugins/mgps-brand-innovacar/includes/shortcode-products.php`
- `app/public/wp-content/plugins/mgps-brand-innovacar/includes/shortcode-page.php`
- `app/public/wp-content/plugins/mgps-brand-innovacar/preview.html`

---

## Adım 1: Hero CTA Butonu Düzeltmesi

**Sorun:** Buton flex stretch nedeniyle tüm container genişliğini kaplıyor.

**Değişiklikler:**
- `.inno-hero__content`'e `align-items: flex-start` ekle
- `.inno-hero__btn` padding'i `16px 40px` → `14px 28px` olarak küçült
- Hem `shortcode-hero.php` hem `preview.html`'de uygula

## Adım 2: Türkçe Karakter Düzeltmesi

**Sorun:** Tüm Türkçe metinlerde ö, ü, ş, ç, ğ, İ karakterleri eksik.

**Değişiklikler:** Tüm shortcode dosyalarında ve preview.html'de Türkçe metinleri düzelt:
- "Surdurulebilir" → "Sürdürülebilir"
- "Urunleri" → "Ürünleri"
- "Cozumleri" → "Çözümleri"
- "Inovasyon" → "İnovasyon"
- "Temizleyiciler" → doğru Türkçe
- Tüm diğer eksik karakterler (ı, İ, ö, Ö, ü, Ü, ş, Ş, ç, Ç, ğ, Ğ)

## Adım 3: Hero Title Renk Ayrımı

**Sorun:** Şu an "CAR" kısmı lime-green.

**Değişiklik:** Logo'ya uygun olarak INNOVA beyaz, CAR outline/siyah alternatif stil. `span` rengini `#D5F206` yerine farklı bir stil yapmak (beyaz INNOVA + koyu/outline CAR).

## Adım 4: Accent Renk Kullanımını Azalt

**Sorun:** Lime-green her yerde kullanılmış. Orijinal sitede minimal.

**Değişiklikler:**
- **Hero scroll indicator:** Lime-green → beyaz/gri tonlara (`rgba(255,255,255,.4)`)
- **Ürün kartı hover link rengi:** `#6B8A00` (lime tonu) → nötr koyu gri veya siyah tonu
- **Mini-title'lar (story, features):** Lime → `rgba(255,255,255,.45)` (dark bg), `rgba(0,0,0,.4)` (light bg) — CLAUDE.md kuralına uygun
- **Counter sayıları:** `#D5F206` → `#fff` beyaz
- **Counter border-top:** `rgba(213,242,6,.12)` → `rgba(255,255,255,.1)`
- **Corner bracket'lar:** Lime tonları korunabilir ama opacity daha düşük

**Korunacak lime accent kullanımları:**
- CTA butonları (hero + category) — orijinal sitedeki gibi
- Feature card hover üst çizgisi
- Feature icon hover arka planı
- Ürün kart hover üst çizgisi
- Dot navigator aktif renk

## Adım 5: Kategori Bazlı Accent Renkleri

**Kullanıcı tercihi:** Her kategori kendi accent rengini kullanacak.

| Kategori | Accent Renk | Kullanım Alanı |
|----------|-------------|----------------|
| Yıkama Şampuanları | `#D5F206` (lime — varsayılan marka rengi) | CTA, hover, mini-title |
| Boya Koruma ve Cila | `#e9ea69` (sarımsı) | Gradient/gölgeleme veya üst başlık rengi |
| Detaylı Temizlik | `#db8901` (turuncu) | Gradient/gölgeleme veya üst başlık rengi |
| Yüzey Hazırlık | `#d0e4e2` (açık turkuaz) | Gradient/gölgeleme veya üst başlık rengi |

**Uygulama:** Category shortcode'unda `accent` parametresi zaten olabilir. Her kategorinin mini-title rengini ve/veya koyu bg'de hafif gradient tint'ini bu renklerle özelleştir.

## Adım 6: Lime Gradient Theme Kaldırma + Dark/Light Alternatif

**Sorun:** `inno-cat--lime` orijinal sitede yok.

**Değişiklikler:**
- `inno-cat--lime` CSS class'ını ve stillerini kaldır
- Kategoriler sırayla dark (#0A0A0A) ve light (#F7F7F7) arka planla değişecek
- Detaylı Temizlik (şu an lime) → dark olacak
- Category shortcode'undan `lime` theme seçeneğini kaldır

## Adım 7: Feature Icon Border-Radius

**Sorun:** `border-radius: 12px` — orijinal site sharp corner.

**Değişiklik:** `.inno-features__icon` border-radius'u `12px` → `4px`

## Adım 8: Product Card Border-Radius Tutarlılığı

**Sorun:** Ürün kartlarında `border-radius: 4px`, diğer elementler sharp.

**Değişiklik:** `.inno-prod__card` border-radius'u `4px` → `2px` (minimal ama tutarlı)

---

## Adım 9: Kategori Accent Renk Kontrastı Düzeltmesi (YENİ)

**Sorun:** Kategori mini-title renkleri, watermark'lar ve accent tonlar okunmuyor.

**Değişiklikler:**

### Hero CAR
- `text-stroke` outline → `color: rgba(255,255,255,.25)` veya koyu/yarı-şeffaf fill

### Mini-title Renk Kontrastı
| Kategori | BG | Eski | Yeni |
|----------|-----|------|------|
| Yıkama | Dark | #D5F206 op:.8 | #D5F206 op:1.0 |
| Koruma | Light | #e9ea69 op:.7 | **#b8a800** (koyu sarı) |
| Detay | Dark | #db8901 op:.8 | #db8901 op:1.0 |
| Yüzey | Light | #d0e4e2 op:.8 | **#5a8a86** (koyu turkuaz) |

### Watermark Opacity
| BG | Eski | Yeni |
|----|------|------|
| Dark | rgba(255,255,255,.02) | Accent bazlı rgba(accent,.04-.05) |
| Light | rgba(0,0,0,.02) | rgba(0,0,0,.04) |

### Uygulama Detayları
- preview.html: inline style ile her kategoriye özel renk
- shortcode-category.php: `accent` parametresinden mini_clr ve watermark rengini hesapla, opacity kaldır
- shortcode-page.php: Koruma accent="#b8a800", Yüzey accent="#5a8a86" olarak güncelle

---

## Adım 10: Marka Sayfası Tasarım Rehberi Oluşturma (YENİ)

**Amaç:** Yeni marka sayfası tasarlarken kullanılacak kapsamlı bir şablon/checklist dokümanı.

**Dosya:** `docs/marka-sayfasi-tasarim-rehberi.md`

**İçerik Yapısı:**
1. Marka Araştırma Aşaması (orijinal site analizi, branding çıkarımı)
2. Renk Stratejisi (nötr yaklaşım kuralları, accent kullanım limitleri)
3. Tipografi Kuralları (font seçimi, weight sınırlamaları, size hiyerarşisi)
4. Section Bazlı Tasarım Rehberi (Hero, Story, Features, Category, Products)
5. WordPress/Elementor Entegrasyon Kuralları (breakout, padding, responsive)
6. Erişilebilirlik ve Performans Checklist
7. Kalite Kontrol Checklist (Türkçe karakter, kontrast, responsive test)

---

## Doğrulama

1. `preview.html`'i tarayıcıda aç — tüm section'ların doğru göründüğünü kontrol et
2. Türkçe karakterlerin tümünün düzgün render edildiğini doğrula
3. Hero butonunun compact boyutta olduğunu kontrol et
4. Kategori arka planlarının dark/light alternatif sırada olduğunu doğrula
5. Accent renklerin sadece CTA, hover ve minimal noktalarda kullanıldığını kontrol et
6. Responsive breakpoint'lerde (1200px, 960px, 768px, 480px) layout'un düzgün çalıştığını kontrol et
7. Plugin PHP dosyalarının preview.html ile birebir eşit CSS/HTML ürettiğini doğrula
8. Kategori mini-title'ların okunabilirliğini kontrol et (WCAG AA kontrast oranı)
9. Watermark'ların silik ama fark edilir olduğunu doğrula
