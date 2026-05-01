# Plan: Badge Absolute Konuma Taşıma — shortcode-hero.php

## Dosya
`/Users/projectx/Local Sites/mgpolishing/app/public/wp-content/plugins/mgps-brand-fraber/includes/shortcode-hero.php`

## Değişiklikler

### 1. CSS `.fb-hero__badge` — satır 41
**Eski:**
```
.fb-hero__badge{display:inline-flex;align-items:center;gap:10px;margin-bottom:28px;padding:8px 16px;background:rgba(255,255,255,.08);backdrop-filter:blur(6px);border:1px solid rgba(255,255,255,.1)}
```
**Yeni:**
```
.fb-hero__badge{position:absolute;bottom:40px;right:180px;z-index:5;display:inline-flex;align-items:center;gap:8px;padding:6px 12px;background:rgba(255,255,255,.06);backdrop-filter:blur(6px);border:1px solid rgba(255,255,255,.08)}
```

### 2. CSS `.fb-hero__badge-flag` — satır 42
**Eski:** `width:24px;height:16px`
**Yeni:** `width:20px;height:13px`

### 3. CSS `.fb-hero__badge-text` — satır 47
**Eski:** `font-size:11px;font-weight:700;letter-spacing:3px`
**Yeni:** `font-size:10px;font-weight:700;letter-spacing:2px`

### 4. Responsive 1200px — satır 68-71
`.fb-hero__badge{right:80px}` ekle

### 5. Responsive 768px — satır 72-78
`.fb-hero__badge{right:30px;bottom:80px}` ekle

### 6. Responsive 480px — satır 79-84
`.fb-hero__badge{right:20px;padding:5px 10px}` ve `.fb-hero__badge-text{font-size:9px;letter-spacing:1.5px}` ekle

### 7. HTML — badge taşıma
- `.fb-hero__content` içinden (h1 sonrası, satır 112-116) badge bloğunu kaldır
- `<div class="fb-hero__scroll"` öncesine (satır 125'in üstü) yerleştir
