# Plan: Projeyi Temizle, Commit & Push

## Context

MTS Kimya Chatbot projesinde 2 aylık iş (Scripts/, output/, Botpress/, assets/ rehberleri, 6 tablolu CSV mimarisi) hâlâ tek bir `first commit` üzerinde bekliyor. Kullanıcı durum özeti istedi ve ardından **"güncel son halini commitle, bazı dosyalar duplicate olabilir, düzenlenmiş son versiyonları commit push yapalım"** dedi.

Proje aşama olarak Botpress entegrasyonunun **Adım 6** noktasında — veri katmanı tamam, bot tarafında ürün kartı/carousel/format iyileştirmeleri kaldı. Bu turda bot tarafına dokunulmayacak, sadece **repo hijyeni + commit/push** yapılacak.

Keşif bulguları:
- Git remote mevcut (GitHub origin) → push edilebilir.
- Kökte duplicate script var (`generate_search_text.py`), Scripts/ içindeki `generate_search_index_raw.py`'dan farklı ama eski kopyası.
- `.gitignore` yok; `.DS_Store` dosyaları track ediliyor.
- Scripts/ altında `.bak` arşiv dosyası mevcut.
- `deepthink.md` Menzerna React sitesi notları — bu projeyle ilgisiz.
- `assets/logs`, `assets/test`, `assets/test2` klasörleri geçici test binary çıktısı.
- `Product Groups/products_fraber.json` ve `products_little_joe.json` geçerli, brand-spesifik veri — tutulacak.

## Temizlik Adımları

### 1. `.gitignore` oluştur
Yeni dosya: [.gitignore](.gitignore)
```
# macOS
.DS_Store
**/.DS_Store

# Python backup
*.bak
*.pyc
__pycache__/

# Geçici test çıktıları
assets/logs/
assets/test/
assets/test2/
```

### 2. Duplicate / eski dosyaları sil
- [generate_search_text.py](generate_search_text.py) — kökteki eski kopya (Scripts/ içindeki güncel sürüm var). **Silinecek.**
- [Scripts/generate_product_content_csv.py.bak](Scripts/generate_product_content_csv.py.bak) — memory notunda "arşivlendi" yazılan backup. **Silinecek.**
- `assets/logs/`, `assets/test/`, `assets/test2/` — binary test çıktıları. **Silinecek** (ignore da edilecek, çift güvence).
- [.DS_Store](.DS_Store) dosyaları → `git rm --cached` ile untrack edilecek (diskten silinmez).

### 3. Kullanıcı onayladı — silinecek
- [deepthink.md](deepthink.md) — Menzerna React sitesi notları, farklı proje. **Silinecek.**
- [convert_to_markdown.py](convert_to_markdown.py), [add_barcodes.py](add_barcodes.py), [chatbot_prompt.txt](chatbot_prompt.txt) — zaten `git status`'ta `deleted:` olarak görünüyor. Commit 1'de silme işlemi stage edilecek.

## Commit Stratejisi

Kullanıcı onayladı: **4 mantıksal commit** (conventional commits).

| # | Commit başlığı | İçerik |
|---|---|---|
| 1 | `chore: add .gitignore and remove tracked junk` | .gitignore oluştur, .DS_Store untrack, duplicate/bak dosyaları sil, deprecated script'leri sil |
| 2 | `feat(data): 6-table architecture with generate pipeline` | Scripts/ dizini (generate_all.py ve 8 generate script'i), output/csv/ altındaki 6 tablo CSV'si, Product Groups/ yeni JSON'lar |
| 3 | `docs(botpress): integration guides and architecture plan` | Botpress/ klasörü (3 MD), kökteki "Botpress Tablo Mimarisi... Planı.md", assets/adim4-5-6*.md, kb_*, search_*, result_type_*, test*_analiz_raporu.md |
| 4 | `chore(assets): research notes and knowledge base enrichment` | assets/ içindeki ADK araştırma notları, knowledge_base_enriched_top50/ vb. kalanlar |

Her commit öncesi `git status` ile kontrol, commit sonrası `git log --oneline` ile doğrulama.

## Push

Tüm commit'ler lokalde hazır olduktan sonra:
```
git push origin main
```

Remote mevcut olduğu keşifle onaylandı. Force push yapılmayacak.

## Doğrulama

- [ ] `git status` → clean working tree (untracked yok, modified yok)
- [ ] `git log --oneline` → 5 commit görünüyor (first commit + 4 yeni)
- [ ] `ls -la` → `.DS_Store` diskte var ama `git ls-files | grep DS_Store` → boş
- [ ] `git ls-files | grep -E '\.(bak|pyc)$'` → boş
- [ ] `gh repo view --web` (opsiyonel) → push'un remote'ta göründüğünü gör
- [ ] CSV satır sayıları değişmemeli: `wc -l output/csv/*.csv` → products_master 623, faq 2120, vb.

## Kritik Dosyalar

- [.gitignore](.gitignore) — oluşturulacak
- [generate_search_text.py](generate_search_text.py) — silinecek
- [Scripts/generate_product_content_csv.py.bak](Scripts/generate_product_content_csv.py.bak) — silinecek
- [deepthink.md](deepthink.md) — **kullanıcıya sorulacak**
- [Scripts/generate_all.py](Scripts/generate_all.py) — commit 2 ana dosyası
- [output/csv/](output/csv/) — commit 2 veri çıktısı
- [Botpress Tablo Mimarisi_ Nihai Dönüşüm Planı.md](Botpress Tablo Mimarisi_ Nihai Dönüşüm Planı.md) — commit 3

## Kapsam Dışı (Bu Turda Yapılmayacak)

- Botpress'te tablo oluşturma / CSV yükleme
- Autonomous Node instructions güncelleme
- Ürün kartı / carousel / fiyat formatı iyileştirmeleri
- Yeni CSV üretimi veya script değişikliği
- Herhangi bir kod refactoring'i

Bu maddeler bir sonraki oturumda, kullanıcı hangisine odaklanmak istediğini söyledikten sonra ele alınacak.
