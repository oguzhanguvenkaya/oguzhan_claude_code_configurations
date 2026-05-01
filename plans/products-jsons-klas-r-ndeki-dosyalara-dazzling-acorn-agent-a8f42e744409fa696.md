# Faz 2-R Revize — 8 Grup Sub_Type Merge İkinci Tur Gözden Geçirme Planı

## 0. Bağlam ve Amaç

Faz 2 ilk turunda ajan, sub_type merge kararlarını **yüzeysel** verdi — yalnızca sub_type adına bakarak. Fonksiyonel olarak tamamen farklı kategoriler birleştirilmiş:

- `towel_wash` (bez/havlu yıkama deterjanı) → `ph_neutral_shampoo` (araba yıkama şampuanı) — **YANLIŞ grup**
- `rinseless_wash` (susuz yıkama) → `prewash_foaming_shampoo` (agresif ön yıkama köpüğü) — **YANLIŞ senaryo**
- `tire_coating` → `paint_coating` — lastik kaplama ≠ boya kaplama
- `fabric_cleaner_concentrate` hedefine hem kumaş temizleyici hem koruyucu (waterproof) karıştırılmış
- `plastic_dressing` hedefine ahşap koruyucu (wood_protector) eklenmiş

Benim görevim bu kararları ikinci turda **her merge için 5 kriter rubriği** ile DETAYLI gözden geçirmek, şu 8 grubu kapsıyor:

| # | Grup | Merge satır | Not |
|---|------|------|-----|
| 1 | abrasive_polish | 3 | |
| 2 | applicators | 3 | |
| 3 | car_shampoo | 4 | kritik yanlışlar var (towel_wash, rinseless_wash) |
| 4 | ceramic_coating | 13 | EN KRİTİK — çoğu şüpheli |
| 5 | clay_products | 3 | |
| 6 | contaminant_solvers | 5 | içinde 1 template_group taşıma var |
| 7 | interior_cleaner | 11 | |
| 8 | microfiber | 7 | |
| **Toplam** | | **49** | |

Not: Bazı merge satırları aynı source→target çiftini farklı SKU'larla tekrar ediyor (örn. ceramic_coating'te 2x `single_layer_coating→paint_coating`). Karar merge-çifti bazında verilir, APPROVE payload satır-SKU bazında yazılır.

## 1. Çalışma Ortamı Ön-Hazırlık

- Çalışma dizini: `/Users/projectx/Desktop/Claude Code Projects/Products Jsons`
- Microservice: `localhost:8787` — health-check OK (200 döndü)
- Secret: `grep RETRIEVAL_SHARED_SECRET retrieval-service/.env | cut -d= -f2-` — mevcut, okunabiliyor
- Input CSV'leri: `data/consolidation/phase2-<group>-subtype-merge.csv` — 8 grubun hepsi mevcut doğrulandı
- Önceki payload'lar: `phase2-<group>-subtype-merge-payload.json` — referans için mevcut

## 2. Veri Toplama Adımları (Her Grup İçin)

Her grup için:

### 2.1. Gruptaki tüm ürünleri dök
```
SECRET=$(grep RETRIEVAL_SHARED_SECRET retrieval-service/.env | cut -d= -f2-)
for o in 0 200 400; do
  curl -sH "Authorization: Bearer $SECRET" "http://localhost:8787/admin/products?limit=200&offset=$o" \
    | jq '.items[] | select(.templateGroup == "<GROUP>") | {sku, name, baseName, templateSubType, specs}'
done > /tmp/phase2R-<group>-all.json
```

### 2.2. Her merge için source ve target SKU'larının fullDescription'ını çek
Input CSV'den source SKU'yu, target sub'taki mevcut SKU'ları (adım 2.1'den filtreleyerek) belirle; her biri için tek-ürün detay endpointi:
```
curl -sH "Authorization: Bearer $SECRET" "http://localhost:8787/admin/products/<SKU>" \
  | jq '{sku, name, baseName, fullDescription, specs}'
```

### 2.3. Not
- CSV'deki `before` alanındaki kaynak sub_type'ta **başka SKU** var mı kontrol et (gruptaki dump'tan). Bazı kaynak sub'lar 2 SKU barındırabilir (ör. car_shampoo/towel_wash 2 ürün).
- Target sub'taki mevcut ürünleri oku — merge sonrası o sub'ta ne olacak, filter sonucu mantıklı mı?

## 3. Karar Rubriği (5 Kriter)

Her `source_sub → target_sub` çifti için:

| # | Kriter | Onay şartı |
|---|---|---|
| 1 | Primary ürün kategorisi | Kaynak + hedef aynı üst kategori (araba yıkama ≠ bez yıkama, kaplama ≠ parlatma) |
| 2 | Hedef yüzey | Aynı yüzeyi hedefliyor (boya/metal/cam/lastik/plastik — farklı = farklı sub) |
| 3 | Kullanım senaryosu | Aynı iş yapılıyor (ön yıkama ≠ ana yıkama ≠ susuz yıkama ≠ bez yıkama) |
| 4 | Formülasyon ailesi | pH aralığı + içerik uyumlu (asidik ≠ alkali ≠ nötr; abrasif ≠ non-abrasif) |
| 5 | Filter mantığı | "X ara" sonrası listede mantıklı ürünler görünür mü? |

**Karar**:
- **APPROVE** — 5/5 kriter MATCH + %90+ emin
- **REJECT** — 1+ kriter MISMATCH → neden + alternatif öner (kalsın / yeni sub / farklı hedef / group değişikliği)
- **ASK** — %60-89 emin → belirsiz, kullanıcıya soru, alternatifler listele

**Kritik kurallar**:
- Primary kategori ihlali → KESİN REJECT
- Sadece sub_type adından kalkma; `baseName` + `fullDescription` mutlaka oku
- Şüphede REJECT değil ASK (alternatif öner)

## 4. Her Merge İçin Üretilecek İç Analiz Şablonu

```
## <group> / <source_sub> → <target_sub>

**Source ürünler (<n>):**
- SKU1 (brand model): fullDesc 50 kelime özet
- ...

**Target ürünler (<n>):**
- SKUa (brand model): fullDesc 50 kelime özet
- ...

**Kriter değerlendirme:**
1. Primary kategori: MATCH/MISMATCH — neden
2. Hedef yüzey: MATCH/MISMATCH — neden
3. Kullanım senaryosu: MATCH/MISMATCH — neden
4. Formülasyon: MATCH/MISMATCH — neden
5. Filter mantığı: OK/NOT OK — neden

**Karar:** APPROVE / REJECT / ASK

**Alternatif (REJECT/ASK ise):**
- Opt A: kaynak kalsın, benzersiz sub_type
- Opt B: yeni sub_type aç: X
- Opt C: farklı hedef: Y
- Opt D: farklı template_group'a taşı: Z
```

## 5. İşlem Sırası (Gruplar)

Kolaydan zora ilerle:

1. **abrasive_polish** (3) — ısınma
2. **applicators** (3)
3. **clay_products** (3) — form=cloth/disc/mitt ayrımı kritik
4. **microfiber** (7) — suede→glass şüpheli, kit→multi_purpose şüpheli
5. **car_shampoo** (4) — **kritik**: towel_wash + rinseless_wash
6. **contaminant_solvers** (5) — içinde template_group taşıma var (özel)
7. **interior_cleaner** (11) — wood_protector→plastic_dressing, fabric_protector→fabric_cleaner
8. **ceramic_coating** (13) — **en kritik, en son yap**: tire_coating, interior/leather→fabric, PPF/trim/wheel variants

## 6. Çıktı Dosyaları (5 dosya)

### 6.1. `data/consolidation/phase2R-A-approved.csv`
APPROVE olan merge'ler, staging-payload formatı:
```
id,scope,sku,field,before,after,label
phase2R-<grup>-<sku>-1,product,<SKU>,template_sub_type,<old>,<new>,APPROVED: <kısa gerekçe>
```
Bir SKU → 1 satır (aynı source→target 2 SKU ise 2 satır).

### 6.2. `data/consolidation/phase2R-A-approved-payload.json`
Staging-API JSON formatı. `scope=product_field` veya mevcut payload şeması — önceki `phase2-*-subtype-merge-payload.json` örnek alınır.

### 6.3. `data/consolidation/phase2R-A-rejected.md`
REJECT olan merge'ler, Markdown tablolarla + gerekçe + alternatif:
```
## <grup> / <source_sub> → <target_sub>
**Reddedilme nedeni:** Kriter N ihlali — ...
**Alternatif:** Opt A/B/C/D ...
```

### 6.4. `data/consolidation/phase2R-A-questions.md`
ASK olan merge'ler, şu format:
```
## Q<n>: <grup> / <source_sub> → <target_sub> ?

**Kaynak (<n>):** SKU1, SKU2 (1 satır özet)
**Hedef (<n>):** SKUa, SKUb (1 satır özet)

**Belirsizlik:** <%90 emin olmama nedeni>

**Alternatifler:**
- A) ...
- B) ...
- C) ...

**Önerim (zayıf):** A/B/C
```

### 6.5. `data/consolidation/phase2R-A-summary.md`
Özet: grup başına APPROVE/REJECT/ASK sayıları, toplam, en önemli 3-5 bulgu.

## 7. Doğrulama Adımı

APPROVE payload hazır olduğunda preview ile test:
```
curl -sH "Authorization: Bearer $SECRET" -H "Content-Type: application/json" \
  -d @data/consolidation/phase2R-A-approved-payload.json \
  http://localhost:8787/admin/staging/preview | jq '{total, planned, unsupported, skipped}'
```

Kabul kriteri: `total == planned`, `unsupported == 0`, `skipped == 0`.

Eğer unsupported/skipped > 0 → sebebini incele, CSV/payload'ı düzelt, tekrar test et.

## 8. Ön-Hipotezler (Raw okumadan, ilk tur kararlardan)

Bu hipotezler **sadece yönlendirme için** — nihai karar ürün description okuyunca verilecek.

| Grup | Merge | Ön-tahmin | Not |
|---|---|---|---|
| car_shampoo | towel_wash → ph_neutral_shampoo | **REJECT** | Bez yıkama deterjanı ≠ araba şampuanı (primary kategori ihlali) |
| car_shampoo | rinseless_wash → prewash_foaming_shampoo | **REJECT** | Susuz yıkama ≠ ön yıkama köpüğü (use-case ihlali) |
| car_shampoo | ppf_shampoo → ph_neutral_shampoo | ASK | PPF için özel formül olabilir |
| ceramic_coating | tire_coating → paint_coating | **REJECT** | Lastik ≠ boya yüzey |
| ceramic_coating | trim_coating → paint_coating | REJECT/ASK | Plastik trim ≠ boya yüzey |
| ceramic_coating | wheel_coating → paint_coating | ASK | Jant kaplama formülü boya ile yakın olabilir |
| ceramic_coating | interior_coating → fabric_coating | ASK | iç = kumaş+plastik+deri, belirsiz |
| ceramic_coating | leather_coating → fabric_coating | ASK | deri ≠ kumaş ama yakın |
| ceramic_coating | ppf_coating → paint_coating | ASK | PPF üstüne uygulama, formül yakın olabilir |
| ceramic_coating | single_layer_coating → paint_coating | APPROVE (muhtemel) | Alt-varyant |
| ceramic_coating | matte/spray/top_coat/kit → paint_coating | APPROVE (muhtemel) | Uygulama formu/varyant |
| interior_cleaner | wood_protector → plastic_dressing | **REJECT** | Ahşap ≠ plastik yüzey |
| interior_cleaner | fabric_protector → fabric_cleaner_concentrate | **REJECT** | Koruyucu (waterproof) ≠ temizleyici |
| interior_cleaner | wood_cleaner → interior_apc | ASK | Ahşap için APC agresiflik sorusu |
| interior_cleaner | interior_disinfectant → surface_disinfectant | APPROVE (muhtemel) | Aynı fonksiyon |
| interior_cleaner | plastic_cleaner/foam_cleaner → interior_apc | APPROVE (muhtemel) | APC varyantı |
| microfiber | suede_cloth → glass_cloth | ASK | Süet bez cam silmede kullanılır ama süet kendi başına |
| microfiber | kit → multi_purpose_cloth | ASK | Kit'i havluya çevirmek bilgi kaybı |
| microfiber | coating_cloth → buffing_cloth | ASK | Kaplama bezi özel low-lint |
| microfiber | chamois_drying_towel → drying_towel | APPROVE (muhtemel) | Alt-form |
| microfiber | cleaning_cloth → multi_purpose_cloth | APPROVE (muhtemel) | Jenerik |
| microfiber | interior_cleaning_applicator → interior_cloth | APPROVE (muhtemel) | Form farkı |
| contaminant_solvers | iron_remover → wheel_iron_remover (2 SKU) | ASK/REJECT | iron_remover üst aile; yanlış yönde merge olabilir |
| contaminant_solvers | wax_remover → tar_glue_remover | ASK | Solvent ortak ama wax ≠ tar |
| contaminant_solvers | Q2M-PYA4000M group fix (contaminant→ceramic_coating, sub=surface_prep) | APPROVE | Zaten yanlış grup düzeltmesi |
| clay_products | clay_cloth/disc/mitt → clay_pad | ASK/APPROVE | Form farkı specs.format'a taşınabilir mi — yoksa ayrı kalsınlar mı? |
| applicators | scrub_pad/wash_sponge/cleaning_sponge → cleaning_pad | ASK | "Pad" geniş bir şemsiye, OK olabilir ama scrub'ın agresifliği kaybolur |
| abrasive_polish | sanding_paste → heavy_cut_compound | ASK | Zımpara izi giderici ağır pasta mı, ayrı bir kategori mi? |
| abrasive_polish | metal_polish → polish | ASK/REJECT | Metal cilası ≠ boya cilası (yüzey farkı) |
| abrasive_polish | one_step_polish → polish | APPROVE (muhtemel) | Agresiflik farkı specs'e taşınabilir |

Tahminimle kabaca: **~18 APPROVE, ~12 REJECT, ~19 ASK** çıkabilir (49 satır üzerinden).

## 9. Kısıtlar

- **COMMIT YAPMAYACAĞIM**.
- Sadece read + yeni dosya yazma (çıktı 5 dosyası) — mevcut dosyaları değiştirmeyeceğim.
- Tek bir ürün merge'i bile olsa analiz yap (örn. `dispenser_bottle → pump_sprayer` tipinde minör olanlar).
- Tüm `baseName` + `fullDescription` okunacak; sadece sub_type adından karar verilmeyecek.
- Plan mode aktif — planın onayı sonrası uygulanacak.

## 10. Final Rapor Formatı

Plan tamamlandıktan sonra kullanıcıya şu özet sunulacak:

- Grup başına APPROVE / REJECT / ASK sayıları (tablo)
- Toplam APPROVE / REJECT / ASK
- En önemli 3-5 bulgu (hangi merge'ler yanlış çıktı, neden)
- Preview API sonucu (total == planned doğrulandı)
- 4 çıktı dosyasının yolları

Toplam 400-700 kelime hedef.

## 11. Sonraki Adımlar (Plan Onayı Sonrası)

1. Bölüm 2 veri toplama (8 grup × adım 2.1–2.2) — bir grubu tamamlayıp bir sonrakine geç
2. Her merge için bölüm 4 analizini iç olarak yap (final dosyaya gitmeyen düşünce iz bırakma)
3. Kararları 4 çıktı dosyasına yaz (bölüm 6)
4. Bölüm 7 preview doğrulaması
5. Bölüm 10 final raporunu sun
