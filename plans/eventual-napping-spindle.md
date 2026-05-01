# Carousel Ok Navigator — WordPress Tema Override Fix

## Sorun
Plugin'deki ok butonları WordPress teması tarafından override ediliyor:
- Tema `button` elementlerine kendi `padding`, `min-width`, `line-height`, `font-size` veriyor
- Bu yüzden 40x40px olması gereken butonlar geniş dikdörtgen oluyor
- SVG ikonlar buton padding'i içinde kaybolup görünmüyor
- Preview.html'de sorun yok çünkü tema CSS'i yok

## Çözüm
`.fb-prod__arrow` stiline tema override'ını engelleyen reset'ler ekle:

**shortcode-products.php satır 64:**
```css
.{uid} .fb-prod__arrow{
    width:40px;height:40px;
    min-width:40px;max-width:40px;     /* tema override engelle */
    padding:0;                          /* tema padding kaldır */
    margin:0;                           /* tema margin kaldır */
    box-sizing:border-box;
    line-height:1;
    font-size:0;                        /* metin varsa gizle */
    border:1px solid #DAD9D8;background:#fff;cursor:pointer;
    display:inline-flex;align-items:center;justify-content:center;
    transition:border-color .25s ease,background .25s ease
}
```

**Aynı değişiklik preview.html'e de (tutarlılık için)**

## Dosyalar
- `includes/shortcode-products.php` (satır 64)
- `preview.html` (satır 167)
