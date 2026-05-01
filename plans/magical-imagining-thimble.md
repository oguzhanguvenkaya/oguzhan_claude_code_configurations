# CSS Dedup Flag Ekleme — IK + Gyeon Plugin'leri

## Context
IK ve Gyeon plugin'lerinde CSS dedup mekanizması eksik. Shortcode'lar birden fazla çağrıldığında (özellikle category 4x + products 4x) aynı `<style>` bloğu sayfaya tekrar tekrar yazılıyor (~15-20KB gereksiz tekrar). Menzerna'daki `$GLOBALS` flag pattern'ini referans alarak IK ve Gyeon'a ekleyeceğiz.

## Referans Pattern (Menzerna — zaten çalışıyor)
```php
ob_start();
if (empty($GLOBALS['__mz_cat_css'])) {
    $GLOBALS['__mz_cat_css'] = true;
?>
<style>/* CSS bir kez yazılır */</style>
<?php } ?>
<!-- HTML her çağrıda yazılır (doğru) -->
```

## Değişiklikler

### IK Plugin (`mgps-brand-ik/includes/`)

| Dosya | Flag adı | Not |
|-------|----------|-----|
| `shortcode-hero.php` | `$GLOBALS['__ik_hero_css']` | Tek çağrı ama yine de ekle |
| `shortcode-story.php` | `$GLOBALS['__ik_story_css']` | Tek çağrı ama yine de ekle |
| `shortcode-features.php` | `$GLOBALS['__ik_features_css']` | Tek çağrı ama yine de ekle |
| `shortcode-category.php` | `$GLOBALS['__ik_cat_css']` | **4 kez çağrılıyor — kritik** |
| `shortcode-products.php` | `$GLOBALS['__ik_prod_css']` | **4 kez çağrılıyor — kritik** |

### Gyeon Plugin (`mgps-brand-gyeon/includes/`)

| Dosya | Flag adı | Not |
|-------|----------|-----|
| `shortcode-hero.php` | `$GLOBALS['__gy_hero_css']` | Tek çağrı |
| `shortcode-story.php` | `$GLOBALS['__gy_story_css']` | Tek çağrı |
| `shortcode-features.php` | `$GLOBALS['__gy_features_css']` | Tek çağrı |
| `shortcode-category.php` | `$GLOBALS['__gy_cat_css']` | **5 kez çağrılıyor — kritik** |
| `shortcode-products.php` | `$GLOBALS['__gy_prod_css']` | **5 kez çağrılıyor — kritik** |

## Uygulama

Her shortcode dosyasında `ob_start();` satırından sonra, `<style>` tag'ından önce:
```php
ob_start();
if (empty($GLOBALS['__ik_cat_css'])) {
    $GLOBALS['__ik_cat_css'] = true;
?>
<style>/* mevcut CSS aynen kalacak */</style>
<?php } ?>
<!-- mevcut HTML aynen kalacak -->
```

`<style>` bloğunun kapanış `</style>` tag'ından sonra if bloğunu kapat (`<?php } ?>`).
HTML kısmı flag dışında kalacak (her çağrıda render edilmeli).

## Dokunulmayacaklar
- CSS içeriği değişmeyecek
- HTML yapısı değişmeyecek
- JS kodu değişmeyecek
- preview.html dosyaları etkilenmez (statik, PHP yok)

## Doğrulama
- Plugin'in WordPress'te aktif olduğu sayfanın kaynak kodunu görüntüle
- Aynı `<style>` bloğunun sayfada sadece 1 kez yazıldığını doğrula
- Görsel olarak sayfanın aynı göründüğünü kontrol et
