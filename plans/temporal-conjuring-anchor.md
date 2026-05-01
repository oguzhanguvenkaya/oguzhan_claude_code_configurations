# Canlıya Yükleme Öncesi Düzeltme Planı
## Plugin'ler: mgps-brand-fraber, mgps-brand-innovacar, mgps-brand-littlejoe

## Context

Üç marka plugin'i canlıya yüklenecek. Audit sonucunda medya URL'leri ve ürün slug'ları zaten canlıda doğrulanmış durumda; sadece kod seviyesinde HIGH ve MEDIUM bulguları mevcut tasarımı bozmadan düzeltmek gerekiyor.

**Audit özeti:**
- **fraber**: TEMİZ — düzeltme gerekmez.
- **littlejoe**: 1 HIGH (duplicate ID), 1 MEDIUM (yanlış kategoriye düşmüş ürün).
- **innovacar**: 2 HIGH (duplicate ID, inline `@import` font), 1 MEDIUM (CSS değişken esc_attr eksikleri).
- **Diğer sayfaları bozma riski yok**: class prefix'leri izole (`fb-`, `lj-`, `inno-`), global CSS/JS yok, hook'lar scoped.

Hero butonu (`#innovacar-categories`, `#lj-categories`) ilk kategori section'ının ID'sine smooth scroll yapıyor — ID'yi tamamen kaldıramayız. Çözüm: **ilk çağrıda** sabit ID'yi tut, **sonraki çağrılarda** ID'yi boş bırak. Bu yaklaşım anchor link'i kırmaz ve duplicate ID hatasını çözer.

---

## Yapılacak Düzeltmeler

### 1. littlejoe L1 — Duplicate `id="lj-categories"` (HIGH)

**Dosya:** [app/public/wp-content/plugins/mgps-brand-littlejoe/includes/shortcode-category.php:71](app/public/wp-content/plugins/mgps-brand-littlejoe/includes/shortcode-category.php#L71)

**Mevcut:** `<section class="<?php echo $uid; ?>" id="lj-categories">` — 8 kez basılıyor.

**Yeni:** Shortcode fonksiyonunun başına flag kontrolü ekle. İlk çağrıda ID ver, sonrakilerde verme:

```php
// Shortcode fonksiyonunun başında, $uid tanımından sonra:
$first_id = empty($GLOBALS['__lj_cat_first']) ? 'lj-categories' : '';
$GLOBALS['__lj_cat_first'] = true;
```

Ve line 71'deki section:
```php
<section class="<?php echo $uid; ?>"<?php echo $first_id ? ' id="' . $first_id . '"' : ''; ?>>
```

**Etki:** Hero butonu `#lj-categories` → ilk kategoriye (Standart) scroll etmeye devam eder. Diğer 7 kategori ID'siz kalır, HTML valid olur.

---

### 2. innovacar I1 — Duplicate `id="innovacar-categories"` (HIGH)

**Dosya:** [app/public/wp-content/plugins/mgps-brand-innovacar/includes/shortcode-category.php:88](app/public/wp-content/plugins/mgps-brand-innovacar/includes/shortcode-category.php#L88)

**Aynı yaklaşım:**
```php
$first_id = empty($GLOBALS['__inno_cat_first']) ? 'innovacar-categories' : '';
$GLOBALS['__inno_cat_first'] = true;
```

Line 88:
```php
<section class="<?php echo $uid; ?>"<?php echo $first_id ? ' id="' . $first_id . '"' : ''; ?>>
```

**Etki:** Hero butonu `#innovacar-categories` → ilk kategoriye (Yıkama Şampuanları) scroll etmeye devam eder. 4 kategori yerine sadece ilkinde ID kalır.

---

### 3. innovacar I2 — Inline `@import` font → `wp_enqueue_style` (HIGH)

**Dosya 1:** [app/public/wp-content/plugins/mgps-brand-innovacar/includes/shortcode-hero.php:18](app/public/wp-content/plugins/mgps-brand-innovacar/includes/shortcode-hero.php#L18)

**Mevcut:**
```css
@import url('https://fonts.googleapis.com/css2?family=League+Spartan:wght@300;400;500;600;700;800;900&display=swap');
```

**Yeni:** Bu satırı **sil**. Render-blocking @import kalkar.

**Dosya 2:** [app/public/wp-content/plugins/mgps-brand-innovacar/mgps-brand-innovacar.php](app/public/wp-content/plugins/mgps-brand-innovacar/mgps-brand-innovacar.php)

Require bloğundan sonra ekle (fraber'deki desenin birebir aynısı):
```php
/* ── Google Font yükle ── */
add_action('wp_enqueue_scripts', function () {
    wp_enqueue_style(
        'mgps-innovacar-font',
        'https://fonts.googleapis.com/css2?family=League+Spartan:wght@300;400;500;600;700;800;900&display=swap',
        [],
        null
    );
});
```

**Etki:** League Spartan fontu HTML `<head>` içinden standart `<link>` olarak yüklenir, inline `@import`'un render-blocking etkisi kalkar. Tasarım görsel olarak aynı kalır.

---

### 4. innovacar I5 — Category CSS değişken escape (MEDIUM)

**Dosya:** [app/public/wp-content/plugins/mgps-brand-innovacar/includes/shortcode-category.php:43,55,56,68](app/public/wp-content/plugins/mgps-brand-innovacar/includes/shortcode-category.php#L43)

Aşağıdaki echo'ları `esc_attr()` ile sar:
- Line 43: `<?php echo $bg_css; ?>` → `<?php echo esc_attr($bg_css); ?>`
- Line 55: `color:<?php echo $txt; ?>` → `color:<?php echo esc_attr($txt); ?>`
- Line 56: `color:<?php echo $sub_txt; ?>` → `color:<?php echo esc_attr($sub_txt); ?>`
- Line 68: `border-top:2px solid <?php echo ($theme === 'light') ? 'rgba(0,0,0,.08)' : 'rgba(255,255,255,.12)'; ?>` → **zaten sabit string**, dokunma. Aynı satırdaki ikinci `<?php echo ... ?>` de sabit, dokunma.

Bu değişkenler switch-case'ten sabit değer alıyor, gerçek XSS yok — ama esc_attr pattern'i doğru olan.

**Etki:** Yok, değerler sabit; sadece kod güvenlik standardına uygun hale gelir.

---

### 5. littlejoe L3 — Ekran temizleyici sprey'in yerini düzelt (MEDIUM)

**Dosya:** [app/public/wp-content/plugins/mgps-brand-littlejoe/includes/shortcode-page.php](app/public/wp-content/plugins/mgps-brand-littlejoe/includes/shortcode-page.php)

**Mevcut:** `little-joe-ekran-temizleme-spreyi-14-ml-mikrofiberli-arac-led-telefon-tablet-oto-cam-temizleyici` slug'ı line 26'da Kategori 1 Standart carousel'inde.

**Aksiyon:**
1. Line 26'daki Standart slug listesinden bu slug'ı **çıkar** (öncesinde ve sonrasındaki virgülü tek virgüle indir).
2. Line 74'teki Kategori 5 Pump Spray carousel slug listesinin **sonuna** ekle (virgülle ayırarak).

**Etki:** Standart kategorisinden 1 ürün eksilir (30 kalır), Pump Spray kategorisine 1 ürün eklenir (6→7).

---

## Dokunulmayacaklar

Audit'te bulunan ancak **bilerek atlananlar** (user direktifi):

- ❌ Fraber F1 (category CSS dedup flag) — LOW, master shortcode zaten tek kez çağırıyor, çıktıda duplikasyon yok.
- ❌ Hardcoded `https://mgpolishing.com/wp-content/uploads/2026/04/*` URL'leri — user onayladı, canlıda mevcut.
- ❌ Ürün slug doğrulaması — user onayladı, slug'lar sabit ve ürünler canlıda.
- ❌ CSS dedup flag'leri (littlejoe/innovacar) — master shortcode pattern'inde duplikasyon oluşmaz, tasarıma dokunmayan opsiyonel iyileştirme.
- ❌ Global font enqueue (fraber/littlejoe her sayfada font yükler) — perf etkisi minör, işlevsel kırıcı değil.

---

## Doğrulama Adımları

Düzeltmelerden sonra:

1. **PHP syntax check** — her üç plugin için `php -l` ile hata olmadığını doğrula (mümkünse).
2. **WordPress sayfası** — Local WP'de her üç marka sayfasını aç, tarayıcı DevTools:
   - Console'da JS hatası yok
   - HTML validation: `document.querySelectorAll('#lj-categories').length === 1` ve `document.querySelectorAll('#innovacar-categories').length === 1`
   - Network: `League+Spartan` fontu `<head>` içinde `<link rel="stylesheet">` olarak yükleniyor (innovacar)
3. **Hero buton smooth scroll** — "ÜRÜNLERİ KEŞFET" butonuna tıkla, ilk kategoriye smooth scroll ettiğini doğrula (hem littlejoe hem innovacar'da).
4. **Littlejoe Pump Spray carousel** — Kategori 5'te ekran temizleyici sprey ürününün göründüğünü doğrula.
5. **Regresyon** — Gyeon, Menzerna, Flex sayfalarını aç, hiçbirinde regresyon olmadığını doğrula.

---

## Değiştirilecek Dosyalar Özet

| # | Dosya | Değişiklik |
|---|---|---|
| 1 | `mgps-brand-littlejoe/includes/shortcode-category.php` | L1 düzeltme: flag-based ID |
| 2 | `mgps-brand-littlejoe/includes/shortcode-page.php` | L3 düzeltme: slug taşıma |
| 3 | `mgps-brand-innovacar/includes/shortcode-category.php` | I1 + I5 düzeltmeleri |
| 4 | `mgps-brand-innovacar/includes/shortcode-hero.php` | I2 düzeltme: `@import` satırı silme |
| 5 | `mgps-brand-innovacar/mgps-brand-innovacar.php` | I2 düzeltme: `wp_enqueue_style` ekleme |

**Toplam:** 5 dosyada ~10 satır değişiklik. Hiçbiri CSS selector'larını veya HTML yapısını değiştirmiyor — tasarım birebir korunur.
