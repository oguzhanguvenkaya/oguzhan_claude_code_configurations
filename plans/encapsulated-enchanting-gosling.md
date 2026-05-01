# FLEX Kategori Görselleri — URL Değişimi + Border Radius

## Context
Kullanıcı 5 kategori için yeni görsel URL'leri paylaştı ve görsellere 20px border-radius eklenmesini istiyor. Test amaçlı yapılan bir denemedir — eski görsel URL'lerine hızlıca geri dönebilmek için "rollback-safe" bir yapı gerekiyor. Değişikliklerin sadece kategori görselleri + görsel köşe yumuşatması ile sınırlı kalması, diğer plugin bölümlerinin (hero, story, features, products, scroll reveal, a11y) etkilenmemesi kritik.

## Karar Matrisi (Kullanıcıyla Netleşti)
- **URL eşleşmesi**: 5 kategoriye yeni görsel, Rotary + Zımpara eski placeholder'da kalacak
- **Köşe dekoru**: Kırmızı L-şekli dekor korunacak (marka kimliği)

### Kategori → Görsel Eşleşmesi
| # | Kategori | Durum | Yeni URL |
|---|----------|-------|----------|
| 1 | Rotary Polisaj Makineleri | Eski | `flex_category_img.png` (değişmez) |
| 2 | Orbital - DA Polisaj Makineleri | YENİ | `ORBITAL-400X400.jpg` |
| 3 | Aydınlatma | YENİ | `AYDINLATMA-400X400.jpg` |
| 4 | Zımpara Makineleri | Eski | `flex_category_img.png` (değişmez) |
| 5 | Toz Emiciler ve Elektrik Süpürgeleri | YENİ | `TOZ-EMICILER-400X400.jpg` |
| 6 | Aksesuarlar | YENİ | `AKSESUARLAR-400X400.jpg` |
| 7 | Yedek Parça | YENİ | `YEDEK-PARCALAR-400X400.jpg` |

---

## Değişiklikler

### 1. `includes/shortcode-page.php` — URL'ler + Rollback Comment'leri

5 kategori için `$categories` array'inde `image_url` satırı güncellenir. **Eski URL, bir üst satıra PHP comment olarak eklenir** — kullanıcı comment marker'larını swap'layarak saniyeler içinde rollback yapabilir.

**Rollback pattern:**
```php
// ROLLBACK: yeni URL'i yorum satırına al, alttakini aktif et
// 'image_url'  => 'https://mgpolishing.com/wp-content/uploads/2026/04/ORBITAL-400X400.jpg',
'image_url'  => 'https://mgpolishing.com/wp-content/uploads/2026/04/flex_category_img.png', // OLD
```

**Aktif durum (yeni URL):**
```php
// ROLLBACK: alttaki satıra geri dön
// 'image_url'  => 'https://mgpolishing.com/wp-content/uploads/2026/04/flex_category_img.png', // OLD
'image_url'  => 'https://mgpolishing.com/wp-content/uploads/2026/04/ORBITAL-400X400.jpg',
```

Uygulanacak 5 kategori: `Orbital - DA` (satır 54), `Aydınlatma` (satır 63), `Toz Emiciler` (satır 81), `Aksesuarlar` (satır 90), `Yedek Parça` (satır 99).

Rotary (satır 45) ve Zımpara (satır 72) satırlarına **dokunulmaz** — eski placeholder kalır.

### 2. `includes/shortcode-category.php` — Border Radius

Mevcut CSS rule (satır 56):
```css
.fx-cat__img-wrap img{width:100%;height:auto;display:block}
```

Güncellenmiş hali:
```css
.fx-cat__img-wrap img{width:100%;height:auto;display:block;border-radius:20px}
```

**Sadece bu tek değer eklenecek.** Kırmızı L-köşe dekor pseudo-element'leri (`.fx-cat__img-wrap::before`) korunur — dekor `-8px` offsette image'ın dışında durduğu için radius ile çakışmaz, sadece görsel olarak rounded corner'la birlikte farklı bir "accent frame" hissi verir.

### 3. `preview.html` — CSS Parity (Tek Satırlık Radius Ekleme)

CLAUDE.md kuralı: "preview.html ile plugin CSS birebir eşit tutulmalı". Sadece CSS eşitliği için aynı `border-radius:20px` değeri `preview.html`'deki `.fx-cat__img-wrap img` rule'una eklenir. **Preview.html'deki görsel URL'leri değiştirilmez** (halihazırda hardcoded placeholder — test canlı sayfada yapılacak).

---

## Dokunulmayacak Dosyalar (Garanti)
- `mgps-brand-flex.php` — ana plugin altyapısı
- `shortcode-hero.php`
- `shortcode-story.php`
- `shortcode-features.php`
- `shortcode-products.php`
- `flex.md`
- `CLAUDE.md`

## Doğrulama

### Live Test (WordPress)
1. FLEX sayfasını tarayıcıda aç (mgpolishing.com/flex veya local'de `[mgps_flex_page]` shortcode'u yerleştirilmiş sayfa)
2. 5 yeni görselin doğru kategorilerde göründüğünü kontrol et:
   - Orbital → otomotiv orbital polisaj görseli
   - Aydınlatma → LED ışık görseli
   - Toz Emiciler → vakum/süpürge görseli
   - Aksesuarlar → aksesuar görseli
   - Yedek Parça → yedek parça görseli
3. Rotary ve Zımpara'nın eski `flex_category_img.png` ile gözüktüğünü teyit et
4. Tüm 7 kategori görselinde 20px yumuşatılmış köşe olduğunu kontrol et
5. Kırmızı L-köşe dekorunun hâlâ göründüğünü teyit et
6. Responsive kontrol (1200px → 768px → 480px): radius tüm boyutlarda çalışıyor mu?

### Rollback Testi (Acil Dönüş)
Eğer görsellerden biri sorunluysa, `shortcode-page.php`'de ilgili satırdaki comment marker'ları takas et:
```diff
- 'image_url'  => 'https://.../ORBITAL-400X400.jpg',
- // 'image_url'  => '...flex_category_img.png', // OLD
+ // 'image_url'  => 'https://.../ORBITAL-400X400.jpg',
+ 'image_url'  => '...flex_category_img.png', // OLD
```
Sayfayı yenile — eski görsel geri gelir. 5 saniyelik işlem.

---

## Kritik Dosyalar
- `/Users/projectx/Local Sites/mgpolishing/app/public/wp-content/plugins/mgps-brand-flex/includes/shortcode-page.php`
- `/Users/projectx/Local Sites/mgpolishing/app/public/wp-content/plugins/mgps-brand-flex/includes/shortcode-category.php`
- `/Users/projectx/Local Sites/mgpolishing/app/public/wp-content/plugins/mgps-brand-flex/preview.html`
