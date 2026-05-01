# Plan: Git Temizliği — Sadece detailagent İçeriğini Tut

## Context

Git repo root'u `Products Jsons/` dizini ama GitHub'a sadece `Botpress/detailagent/` içeriği push edilmeli. Şu an **3418 dosya tracked**, bunların sadece **39'u detailagent** içinde. Geri kalan 3379 dosya (agents/, output/, Product Groups/, assets/, Scripts/, vb.) git'ten çıkarılmalı.

## Mevcut Durum

| Kategori | Dosya | Durum |
|----------|-------|-------|
| `Botpress/detailagent/` | 39 | ✓ Kalacak |
| `agents/` | 3147 | ✗ Çıkarılacak |
| `output/` | 63 | ✗ Çıkarılacak |
| `knowledge_base_enriched_top50/` | 54 | ✗ Çıkarılacak |
| `Product Groups/` | 26 | ✗ Çıkarılacak |
| `chatbot_md/` | 26 | ✗ Çıkarılacak |
| `assets/` | 25+2 | ✗ Çıkarılacak |
| `Scripts/` | 20 | ✗ Çıkarılacak |
| `Botpress/TABLES/` + `Botpress/docs/` | 7 | ✗ Çıkarılacak |
| Root dosyalar (report*.md, *.py, vb.) | 8 | ✗ Çıkarılacak |

## Plan

### Adım 1: Root `.gitignore` güncelle
`/Users/projectx/Desktop/Claude Code Projects/Products Jsons/.gitignore` dosyasını yeniden yaz:

```gitignore
# ── Her şeyi ignore et, sadece detailagent hariç ──
*

# detailagent dizinini whitelist
!Botpress/
!Botpress/detailagent/
!Botpress/detailagent/**

# detailagent içindeki ignore'lar (.gitignore'un kendi kuralları devralınır)
```

### Adım 2: detailagent `.gitignore` güncelle
Mevcut dosyaya eklenecekler:

```gitignore
# Eval files (generated)
evals/

# Lock files
bun.lock

# Temporary/debug scripts
scripts/check-*.ts
scripts/deep-*.ts
scripts/find-*.ts
scripts/reseed-*.ts
scripts/verify-v72-merged.ts
scripts/count-all-tables.ts

# Experimental table splits
src/tables/product-desc-part*.ts
```

### Adım 3: Tracked dosyaları git'ten kaldır (dosyaları silmeden)
```bash
# detailagent dışındaki TÜM tracked dosyaları untrack et (lokal dosyalar SİLİNMEZ)
git rm --cached -r agents/ output/ knowledge_base_enriched_top50/ Product\ Groups/ chatbot_md/ assets/ Scripts/ Botpress/TABLES/ Botpress/docs/
git rm --cached .gitignore Ajan_Ajans_Manifesto_v2.md "Botpress Tablo Mimarisi_*.md" analyze_product_model.py remove_short_description_column.py report.md report_1.md report_5.md revize.md
```

### Adım 4: Commit ve push
```bash
git add Botpress/detailagent/ .gitignore
git commit -m "chore: clean repo — keep only detailagent, remove 3379 data/asset files"
git push origin main
```

## Kritik Dosyalar

| Dosya | Değişiklik |
|-------|-----------|
| `.gitignore` (repo root) | Yeniden yazılacak — wildcard ignore + detailagent whitelist |
| `Botpress/detailagent/.gitignore` | Eval, bun.lock, debug script'ler eklenecek |
| 3379 tracked dosya | `git rm --cached` ile untrack (lokal dosyalar korunur) |

## Doğrulama

1. `git ls-files` sonrası sadece `Botpress/detailagent/` altındaki dosyalar görünmeli (~39-51 dosya)
2. `git status` temiz olmalı (untracked'lar .gitignore ile filtrelenmeli)
3. Lokal dosyalar (`agents/`, `output/`, `Scripts/` vb.) hala diskte mevcut olmalı
4. `git push` sonrası GitHub'da sadece detailagent dosyaları olmalı
