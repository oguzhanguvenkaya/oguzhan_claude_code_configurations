# MTS Kimya Retrieval Microservice - Phase 4 Bugs Veri Doğrulaması

## Durum
Plan modunda. Script yazılması ve çalıştırılması gerekliyor.

## Özet
6 buğu vardığı bilinen retrieval microservice'inde veri doğrulama sorguları çalıştırılacak:
- Q1: AntiFog ürünleri ve search_text'teki "seramik" kontrolü
- Q2: GYEON 1000 TL altı seramik kaplamalar
- Q3: GYEON Syncro ve ceramic_coating durability rating'leri
- Q4: MENZERNA 400 serisi ve kalın pastaların tümü
- Q5: 1500-2500 TL arası pasta ürünleri ve sizes variants
- Q6: Q2-OLE100M silikon FAQ'ları
- Q7: Q2-OLE100M getRelatedProducts use_with relasyonları

## Adımlar

### 1. Script Hazırlama
`scripts/inspect-phase4-bugs.ts` oluşturulacak:
- DB bağlantısı: `src/lib/db.ts` pattern'i takip
- 7 ayrı sorgu grubunda çalıştırılacak
- Sonuçlar console.log ile dump edilecek

### 2. Çalıştırma
```bash
cd "/Users/projectx/Desktop/Claude Code Projects/Products Jsons/retrieval-service"
PATH="$HOME/.bun/bin:$PATH" bun run scripts/inspect-phase4-bugs.ts
```

### 3. Sonuç Raporlaması
- Her Q için numaralı başlık
- Sonuç tablosu (markdown)
- Boş sonuçlar: "EMPTY"
- Uzunluk limiti yok, tüm veriler rapor edilecek

## Teknik Detaylar
- postgres.js paketi: sql.unsafe() veya sql template literals kullanımı
- .env SUPABASE_DB_URL kullanılacak
- Type-safe olmayan raw SQL da sorun değil (inspect script için)
- sql.end() ile cleanup yapılacak

