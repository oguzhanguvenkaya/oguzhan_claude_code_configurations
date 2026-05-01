# Plan: shortcode-story.php Düzenlemeleri

## Dosya
`/Users/projectx/Local Sites/mgpolishing/app/public/wp-content/plugins/mgps-brand-fraber/includes/shortcode-story.php`

## Değişiklikler

### 1. Visual'e order:2 (satır 39)
- Eski: `.fb-story__visual{position:relative}`
- Yeni: `.fb-story__visual{position:relative;order:2}`

### 2. Content'e order:1 (satır 46)
- Eski: `.fb-story__content{display:flex;flex-direction:column;gap:24px}`
- Yeni: `.fb-story__content{display:flex;flex-direction:column;gap:24px;order:1}`

### 3. Mini-title rengi düzeltmesi (satır 47)
- Eski: `color:#004C94` (#001a33 bg üzerinde kötü kontrast)
- Yeni: `color:#5BA3D9` (açık mavi, okunabilir)

### 4. 960px breakpoint kontrolü (satır 65)
- `.fb-story__visual{max-width:500px;margin:0 auto}` mevcut
- Single-column'da order değerleri geçerli kalır: content üstte (order:1), visual altta (order:2)
- Ekstra override gerekmiyor
