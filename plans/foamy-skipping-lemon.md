# Stok Giriş/Çıkış Veri Analizi — Google Sheets Eğitim Projesi

## İlerleme Durumu
- [x] FAZ 0: Setup & Veri İthalat — TAMAMLANDI
  - Google Sheets bağlantısı (MCP + gspread) çalışıyor
  - Spreadsheet ID: 1lk-nXptO9IRBugH-UF7sxOtkL5X31twLjXGaN_UY6Kc
  - RAW_import (955 satır), RAW_stockout (19,201 satır), order_data yüklendi
- [x] FAZ 1 — DEVAM EDİYOR
  - CLEAN_import QUERY tamamlandı (813 satır, gereksiz sütunlar çıkarıldı: Depo Adı, Alt Grubu, Cari Kodu)
  - Tarih sütunu (I) ARRAYFORMULA ile eklendi, format düzeltildi
  - Ay sütunu (J) ARRAYFORMULA+TEXT ile eklendi
  - **SIRADA**: CLEAN_stockout QUERY + Tarih + Ay sütunları eklenmeli, sonra Ocak 2025 filtresi
- [ ] FAZ 2: Stok giriş sıklığı analizi
- [ ] FAZ 3: Stok erime analizi
- [ ] FAZ 4: Sipariş optimizasyonu

## Context

Kullanıcı oto detailing ürünleri ithal eden bir işletme yönetiyor. 14 aylık stok giriş verisi (import_data.xlsx) ve stok çıkış/satış verisi (stock_out_data.XLS) mevcut. Amaç: ürün ve marka bazında stok giriş sıklığını, stok erime hızını analiz edip optimal sipariş miktarı ve zamanlamasını belirlemek. Kullanıcı bir veri analizi kursunda ve **tüm analizi Google Sheets** ile yapacak — bu bir öğrenme projesi.

## Veri Özeti

| Dosya | Satır | Benzersiz Barkod | Barkod=NaN | Marka Sayısı | Tarih Aralığı |
|-------|-------|------------------|------------|--------------|---------------|
| import_data.xlsx | 954 | 421 | 141 | 16 | Oca 2025 – Şub 2026 |
| stock_out_data.XLS | 19,200 | 733 | 981 | 21 | Ağu 2024 – Ara 2025 |
| order_data.XLS | 2,154 | — | — | — | Oca 2024 – devam |

**Ortak sütunlar**: Barkodu, Stok Adı, Temel Mik., KPB Fiyatı, Grubu, Markası, Kayıt Tarihi
**Ek (stock_out)**: Müşteri bilgileri (Cari Kodu, Ticari Unvanı, İl, İlçe)
**Parametre**: Kargo süresi ortalama ~2 ay (1-3 ay arası)

## Yaklaşım: Tek Workbook, Faz Bazlı Sheet'ler

### Sheet Yapısı
```
RAW_import      → Ham veri (salt okunur, sarı arka plan)
RAW_stockout    → Ham veri (salt okunur, sarı arka plan)
CLEAN_import    → Temizlenmiş giriş verisi
CLEAN_stockout  → Temizlenmiş çıkış verisi (Oca 2025+)
PIVOT_entry     → Giriş sıklığı pivot tabloları
PIVOT_depletion → Stok erime analizi
DASH_brand      → Marka bazlı dashboard + grafikler
DASH_reorder    → Sipariş önerileri + senaryo analizi
REF_notes       → Metodoloji notları ve öğrenme günlüğü
```

---

## Uygulama Planı

### FAZ 0: Setup & Veri İthalat
**Öğrenme**: Veri organizasyonu, "ham veri kutsaldır" prensibi

1. Google Sheets workbook oluştur, yukarıdaki sheet'leri aç
2. `import_data.xlsx` → `RAW_import` olarak import et
3. `stock_out_data.XLS` → `RAW_stockout` olarak import et
4. RAW sheet'lere sarı arka plan + "DÜZENLEME" uyarısı ekle
5. `REF_notes`'a veri sözlüğü yaz (sütun adları, anlamları, veri tipleri)

### FAZ 1: Veri Temizleme
**Öğrenme**: QUERY, FILTER, DATE fonksiyonları, COUNTIF ile doğrulama

**`CLEAN_import` sheet'i:**
- QUERY ile barkodu boş satırları filtrele: `=QUERY(RAW_import!A:K, "SELECT * WHERE B IS NOT NULL")`
- Yeni sütun `Tarih_Clean`: `=DATE(MID(K2,7,4), MID(K2,4,2), LEFT(K2,2))` (saat kısmı yok)
- Yardımcı sütunlar: `Ay` → `=TEXT(Tarih_Clean, "YYYY-MM")`, `Hafta` → `=WEEKNUM(Tarih_Clean)`
- Doğrulama: COUNTBLANK, satır sayısı kontrolü (~813 satır bekleniyor)

**`CLEAN_stockout` sheet'i:**
- Aynı temizleme + tarih filtresi: sadece Ocak 2025 sonrası
- QUERY: `"SELECT * WHERE B IS NOT NULL AND ...tarih filtresi..."`
- Çapraz doğrulama: stock_out'taki kaç barkod import'ta da var? (COUNTIF)

### FAZ 2: Stok Giriş Sıklığı Analizi
**Öğrenme**: COUNTIFS/SUMIFS (manuel pivot), Pivot Table, grafik oluşturma, koşullu biçimlendirme

**`PIVOT_entry` sheet'i:**

1. **Manuel pivot** — marka × ay matrisi, COUNTIFS ile doldur
   - Bu, pivot table'ın arka planda ne yaptığını gösterir
2. **Google Sheets Pivot Table** — aynı veriyi otomatik pivot ile oluştur, karşılaştır
3. **Ürün bazlı sıklık** — hangi barkodlar en sık giriş yapıyor? Top 20 liste
4. **Görselleştirme**:
   - Bar chart: marka başına toplam giriş
   - Stacked bar: marka × ay (mevsimsellik görünür)
   - Koşullu biçimlendirme: renk skalası (beyaz→koyu yeşil)
5. **Değer analizi**: SUM(KPB Fiyatı × Temel Mik.) marka ve ay bazında

### FAZ 3: Stok Erime Analizi
**Öğrenme**: VLOOKUP/INDEX-MATCH, tarih aritmetiği, zaman serisi düşüncesi

**`PIVOT_depletion` sheet'i:**

1. **Ürün zaman çizelgesi** — her barkod için:
   - Toplam giriş miktarı (SUMIFS → CLEAN_import)
   - Toplam çıkış miktarı (SUMIFS → CLEAN_stockout)
   - Net stok = Giriş - Çıkış
2. **Aylık çıkış hızı** — barkod × ay pivot: aylık kaç adet çıkış?
   - Ortalama aylık çıkış = `AVERAGEIFS(aylık çıkışlar, >0)`
3. **Stok erime süresi tahmini**:
   - Tahmini ay = Net Stok / Ort. Aylık Çıkış
   - Net Stok < 2 aylık çıkış → **UYARI** (kargo süresi 2 ay)
4. **Marka özeti** — marka bazında ortalama ve minimum erime süresi
5. **Görselleştirme**:
   - Scatter plot: X = aylık çıkış, Y = net stok (kadranlı analiz)
   - Line chart: aylık çıkış trendi marka bazında

### FAZ 4: Sipariş Optimizasyonu
**Öğrenme**: IF/IFS, Data Validation (dropdown), parametrik modelleme

**`DASH_reorder` sheet'i:**

1. **Parametre hücreleri** (dropdown ile):
   - Kargo süresi: 1 / 1.5 / 2 / 2.5 / 3 ay
   - Güvenlik faktörü: 1.0 / 1.25 / 1.5 / 2.0
2. **Yeniden sipariş noktası**: `= Ort_Aylık_Çıkış × Kargo_Süresi × Güvenlik_Faktörü`
3. **Sipariş tablosu**: Barkod | Ürün | Marka | Net Stok | Ort. Çıkış | Reorder Point | Sipariş Gerekli? | Önerilen Miktar
4. **Koşullu biçimlendirme**:
   - 🔴 Net stok < 1 aylık çıkış = KRİTİK
   - 🟡 Net stok < 2 aylık çıkış = UYARI
   - 🟢 Net stok ≥ 3 aylık çıkış = SAĞLIKLI
5. **Marka bazında sipariş özeti**: toplam miktar ve tahmini maliyet

### FAZ 5 (Sonra): order_data.XLS Entegrasyonu
- 2D yapı incelenecek, pivot table'a uygun hale getirilecek
- İptal edilmiş siparişler filtrelenecek
- Gerçek sipariş-teslim süresi hesaplanarak lead time varsayımı rafine edilecek

---

## Kullanılacak Agent ve Skill'ler

| Faz | Agent/Skill | Kullanım |
|-----|-------------|----------|
| Tüm fazlar | `data-analyst` | Google Sheets formül rehberliği, dashboard tasarımı |
| Faz 3-4 | `data-scientist` | İstatistiksel analiz yöntemleri, pattern recognition |
| Faz 4 | `ecommerce-analyst` | Sipariş optimizasyonu, stok yönetimi iş perspektifi |
| Faz 5 | `business-analyst` | Order data yapı analizi ve iş süreci haritalama |
| Gerektiğinde | `trend-analyst` | Mevsimsellik ve trend tespiti |

---

## Google Sheets Teknik Yol Haritası

| Faz | Teknik | Neden Önemli |
|-----|--------|-------------|
| 0 | Sheet isimlendirme, hücre biçimlendirme | Organizasyon alışkanlığı |
| 1 | QUERY, FILTER, DATE, TEXT, MID, LEFT | Veri temizleme temeli |
| 1 | COUNTIF, COUNTBLANK | Kalite kontrol |
| 2 | COUNTIFS, SUMIFS (manuel pivot) | Aggregation mantığını anlamak |
| 2 | Pivot Table | Otomasyonu anlamak |
| 2 | Bar/stacked bar chart, koşullu biçimlendirme | Görsel iletişim |
| 3 | VLOOKUP / INDEX-MATCH | Çapraz veri eşleştirme |
| 3 | MINIFS, MAXIFS, AVERAGEIFS | Koşullu istatistik |
| 3 | Scatter plot, line chart | Pattern tanıma |
| 4 | IF/IFS, iç içe mantık | Karar çerçeveleri |
| 4 | Data Validation (dropdown) | Parametrik analiz |
| 4 | Named Ranges | Formül okunabilirliği |

## Performans Notları
- stock_out 19,200 satır — Google Sheets yönetebilir ama dikkat gerekli
- ARRAYFORMULA + VLOOKUP 19k satırda yavaşlar → QUERY tercih et
- Pivot table'lar CLEAN sheet'lerden oluşturulsun, RAW'dan değil
- Gerekirse ön-aggregation yap, sonra aggregated veriye referans ver

## Doğrulama
- Her faz sonunda satır sayısı, benzersiz barkod sayısı kontrol edilecek
- Manuel pivot vs. otomatik pivot sonuçları karşılaştırılacak
- Stok erime hesabında negatif stok çıkarsa → veri tutarsızlığı araştırılacak
- Sipariş önerileri bilinen gerçek siparişlerle karşılaştırılarak model doğrulanacak
