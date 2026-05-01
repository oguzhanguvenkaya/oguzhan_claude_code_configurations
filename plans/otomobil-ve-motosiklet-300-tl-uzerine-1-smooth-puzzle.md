# Plan: Trendyol Kampanya Excel Filtreleme (%6 Eşik)

## Context
Kullanıcı, Trendyol kampanya listesi için hazırlanmış bir Excel dosyasında, zorunlu indirim oranı %6'dan yüksek olan ürünleri listeden çıkarmak istiyor. Böylece kampanyaya yalnızca kârlılığı (veya kabul edilebilir indirim oranını) koruyan ürünler dahil olacak.

- **Kaynak dosya:** `/Users/projectx/Desktop/Claude Code Projects/campaign/otomobil-ve-motosiklet---300-tl-uzerine--10-indirim----30-trendyol-karsilamali_2026-04-17_23-35_tr-TR_part_1.xlsx`
- **Sheet adı:** `KampanyaUrunleri`
- **Toplam satır:** 294 (1 başlık + 293 veri)
- **İlgili sütunlar:**
  - `J` = `Mevcut Satış Fiyatı`
  - `K` = `Maksimum Girebileceğin Fiyat`

## Hesaplama Kuralı
Her veri satırı için:
```
fark       = Mevcut Satış Fiyatı - Maksimum Girebileceğin Fiyat
fark_yuzde = (fark / Mevcut Satış Fiyatı) * 100
```
- `fark_yuzde > 6`  → satır **silinir**
- `fark_yuzde ≤ 6`  → satır **kalır** (sınır dahil)

Örnek (ilk 3 veri satırı):
| # | Mevcut | Max | Fark | Fark % | Sonuç |
|---|--------|-----|------|--------|-------|
| 1 | 549.00 | 518.87 | 30.13 | 5.49% | Kalır |
| 2 | 159.00 | 150.12 | 8.88  | 5.58% | Kalır |
| 3 | 549.00 | 517.61 | 31.39 | 5.72% | Kalır |

## Uygulama Adımları
1. `openpyxl` ile workbook'u `data_only=False` olarak aç.
2. `KampanyaUrunleri` sheet'inde 2. satırdan itibaren dolaş.
3. J ve K hücre değerlerini oku. Sayısal değilse (None/metin) satır **silme listesine** eklenir.
4. Yüzde farkı hesapla; %6'dan yüksekse satır indeksini sil listesine ekle.
5. Satırları **tersten** (alttan yukarı) `ws.delete_rows(idx)` ile sil — indeks kayması olmasın diye.
6. Workbook'u kaydet (format, başlık stili ve diğer sütunlar aynen korunur).
7. Özet raporla: kaç satır silindi, kaç satır kaldı.

## Kritik Dosyalar
- **Okunacak/yazılacak:** yukarıdaki `.xlsx` dosyası
- **Yazılacak script:** geçici `/tmp/filter_campaign.py` (tek seferlik işlem, projeye kod eklenmez)

## Kullanıcı Kararları (onaylandı)
1. **Çıktı:** Orijinal dosyanın üzerine yazılacak (yedek alınmayacak).
2. **Eksik fiyat:** J veya K hücresi boş/sayısal değilse satır silinecek.

## Doğrulama
- İşlem sonrası: `python3 -c "import openpyxl; wb=openpyxl.load_workbook('<dosya>'); ws=wb['KampanyaUrunleri']; print('kalan satır:', ws.max_row - 1)"`
- Rastgele 2-3 kalan satır için manuel yüzde hesabı yap; hepsi ≤ %6 olmalı.
- Silinmesi beklenen örnek satırlardan birini (önceden not edilmiş) eski/yeni dosyada karşılaştır.
