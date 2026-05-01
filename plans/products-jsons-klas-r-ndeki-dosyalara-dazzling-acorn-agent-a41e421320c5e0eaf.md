# Phase 2 Revize — B Ajan Planı
## MTS Kimya Katalog Konsolidasyonu — 7 Grup Sub-Type Merge Derin İncelemesi

**Durum:** PLAN (okuma tamam, yazma için onay bekliyor)
**Kapsam:** 7 grup × toplam 27 merge kararı
**Microservice:** localhost:8787 (read-only GET'ler zaten yapıldı)

---

## Özet Karar Tablosu (Plan)

| Grup | APPROVE | REJECT | ASK | Toplam |
|------|---------|--------|-----|--------|
| 1. paint_protection_quick | 5 | 0 | 0 | 5 |
| 2. polisher_machine | 3 | 5 | 3 | 11 |
| 3. polishing_pad | 0 | 2 | 0 | 2 |
| 4. spare_part | 2 | 3 | 0 | 5 |
| 5. sprayers_bottles | 0 | 1 | 0 | 1 |
| 6. storage_accessories | 1 | 0 | 1 | 2 |
| 7. tire_care | 0 | 0 | 3 | 3 |
| **TOPLAM** | **11** | **11** | **7** | **29*** |

\* polisher_machine'de 11 satır var; iki satır (corded/cordless) ayrı ayrı değerlendiriliyor.

---

## 1. paint_protection_quick — 5 Merge (Hepsi APPROVE)

### Kaynak merge'ler
- `spray_wipe_sealant` → `spray_sealant` (SKU: 75182, 74059, 700096)
- `spray_rinse_sealant` → `spray_sealant` (SKU: Q2M-WCYA4000M, 79304)

### Rubrik
| # | Kriter | Durum |
|---|--------|-------|
| 1 | Primary kategori (Wax ve Sprey Cilalar / Boya Koruma) | Tüm ürünler aynı | OK |
| 2 | Hedef yüzey (araç boyası) | Aynı | OK |
| 3 | Kullanım senaryosu (yıkama sonrası hızlı koruma) | Aynı | OK |
| 4 | Formülasyon (SiO²/wax/sealant sprey) | Aile aynı; rinse/wipe uygulama adımı ayrımı specs.application_method'ta zaten var (`Spray & Wipe`, `Spray & Rinse`) | OK |
| 5 | Filter mantığı | Kullanıcı "sprey cila" diye arar; wipe vs rinse **uygulama adımı**, ürün kategorisi değil | OK |

### Kanıt
- Hedef sub_type `spray_sealant` zaten 5 ürün barındırıyor (Q2M-CMR500M, Q2M-CRYA250M, Q2M-PPFMR500M, 700097, 26919.271.001). Merge sonrası 10 ürünlük sağlıklı bucket olur.
- specs.application_method alanı rinse vs wipe ayrımı için yeterli (zaten mevcut).

### Karar: **APPROVE (5/5)**
- phase2R-B-approved.csv'e 5 satır.
- payload: `template_sub_type` update, scope=product.

---

## 2. polisher_machine — 11 Merge (Karışık)

### 2.1. corded_rotary_polisher → rotary_polisher (SKU 373680, 406813, SGGF179)
### 2.2. cordless_rotary_polisher → rotary_polisher (SKU 533019)

**Rubrik**
| # | Kriter | Durum |
|---|--------|-------|
| 1 | Primary kategori | Her ikisi de rotary polisaj makinesi | OK |
| 2 | Hedef yüzey | Aynı (araç boyası) | OK |
| 3 | Kullanım senaryosu | Aynı (ağır kesim polisaj) | OK |
| 4 | Formülasyon/yapı | Aynı motion; güç kaynağı farklı | Kısmen |
| 5 | Filter mantığı | Kullanıcı "kablosuz" ister mi? `specs.power_source` zaten dolu ("Corded", "Battery") ve filter için yeterli | Dikkat |

**Risk:** Eğer bot/filter UI `sub_type` üzerinden gidiyorsa "kablosuz rotary" araması erişim kaybeder. Ama `specs.power_source` kırılımı hâlâ var; retrieval katmanı bu alanı facet olarak kullanabilir.

**Karar:** **ASK** — `specs.power_source` filter'ı bot tarafında kullanılıyor mu? Evet ise APPROVE (4 satır); hayır ise corded ve cordless ayrı kalsın.

**Soru (questions.md Q1):** corded/cordless ayrımı `specs.power_source` ile retrieval'da yapılabiliyor mu? Birleştirme bot filter'ını bozar mı?

### 2.3. forced_rotation_polisher → da_polisher (SKU 418072, 533020)

**Rubrik**
| # | Kriter | Durum |
|---|--------|-------|
| 1 | Primary kategori | Her ikisi de orbital polisaj | OK |
| 2 | Hedef yüzey | Aynı | OK |
| 3 | Kullanım senaryosu | DA = random orbital; forced rotation = **pozitif sürüşlü** DA (Gear Driven Dual Action) | Kısmen |
| 4 | Formülasyon/yapı | İsimlerden (FLEX XCE "Pozitif Sürüş" vs FLEX XFE "Random Orbital"). **Mekanik olarak farklı**: forced rotation ped dönüşünü zorlar (hologram yapmaz, yüksek kesim); pure DA serbest orbital. Teknik olarak farklı araç tipleri | İHLAL |
| 5 | Filter mantığı | Profesyonel kullanıcı "DA" ile "forced rotation / gear driven DA" arasında ayrım ister | İHLAL |

**Mevcut `da_polisher` bucket:** 533021, 447129, 418080, SGGF181 (4 ürün, hepsi pure random orbital)

**Karar:** **ASK** — kriter 4-5 ihlal ama tek bir alternatif için 2 ürün yeterli mi?

**Soru (questions.md Q2):** forced_rotation_polisher ayrı mı kalsın?
- A) Ayrı kalsın (2 ürünle tek başına sub)
- B) da_polisher altına merge, ayrım `specs.drive_type=gear_driven`
- C) Yeni sub `gear_driven_da_polisher`
- **Önerim (zayıf): B** — specs.drive_type ile ayır, sub_type'ı birleştir.

### 2.4. other → polisher_accessory (SKU 532579, 516112, SGGS003, SGGC086, SGGC055)

**Kritik bulgu:** `other` altındaki 5 üründen sadece 1'i (516112 "FLEX FS 140 PXE 80 Esnek Uzatma") gerçekten polisher aksesuarı.

| SKU | Ürün | Gerçek sınıf |
|-----|------|---------------|
| 532579 | FLEX HG 650 Sıcak Hava Tabancası 2000W | Heat gun — polisher DEĞİL |
| SGGS003 | SCCB Kısa Nozul Hava Tabancası — Air Blow Gun | Hava üfleyici — polisher DEĞİL |
| SGGC086 | SGCB Air Blow Gun Yüksek Basınçlı Hava Tabancası | Hava üfleyici — polisher DEĞİL |
| SGGC055 | SGCB Tornador Detaylı Temizlik Tabancası | Interior cleaning tool — polisher DEĞİL |
| 516112 | FLEX FS 140 PXE 80 Esnek Uzatma | **Gerçek polisher aksesuarı** |

**Rubrik (her biri için)**

| SKU | K1 (kategori) | Karar |
|-----|---------------|-------|
| 532579 | Heat gun ≠ polisher | **REJECT**: yeni template_group `heat_gun` veya `other_tools` |
| SGGS003 | Air blower ≠ polisher | **REJECT**: yeni tg `air_blower` |
| SGGC086 | Air blower ≠ polisher | **REJECT**: yeni tg `air_blower` |
| SGGC055 | Interior cleaning gun ≠ polisher | **REJECT**: mevcut `interior_cleaner` grubu uygun |
| 516112 | Polisher extension | **APPROVE** → polisher_accessory |

**Karar özeti:** 1 APPROVE, 4 REJECT.

**REJECT notu (rejected.md):** Bu 4 SKU phase 1/2'de yanlış template_group'a atanmış. Phase 3'te tg-merge/taxonomy-fix gerekli. Şimdi `other` altında kalsınlar.

---

## 3. polishing_pad — 2 Merge (İkisi de REJECT)

### 3.1. felt_pad → wool_pad (SGGA081)

**SGGA081:** "SGCB **Cam Çizik Giderici** Keçe 150mm — Silecek İzi Kireç Leke Giderici"
- `material: Rayon/Keçe`
- `target_surface: Glass`
- `cut_level: Glass Cut`

**Wool pad bucket (5 ürün):** Hepsi "Kuzu Yünü Polisaj Keçesi" — araç boyası için.

**Rubrik**
| # | Kriter | Durum |
|---|--------|-------|
| 1 | Primary kategori | Cam polisaj ≠ boya polisaj | İHLAL |
| 2 | Hedef yüzey | Glass vs Paint | İHLAL |
| 4 | Malzeme | Rayon/keçe ≠ Kuzu yünü | İHLAL |

**Karar:** **REJECT**
**Alternatif:** Yeni sub_type `glass_polishing_pad` (tek ürün ama semantik doğru) — veya `felt_pad`'i koru.

### 3.2. microfiber_pad → foam_pad (NPMW6555)

**NPMW6555:** "MG PS Ara Kesim Pasta **Keçesi**" — `material: Mikrofiber`, `cut_level: Medium Cut`

**Foam pad bucket (24 ürün):** Hepsi "Sünger" / Foam.

**Rubrik**
| # | Kriter | Durum |
|---|--------|-------|
| 1 | Primary kategori | Her ikisi de boya polisaj pedi | OK |
| 2 | Hedef yüzey | Araç boyası | OK |
| 4 | Malzeme | Mikrofiber ≠ Köpük — **kesim karakteri, ısı davranışı, ömür farklı** | İHLAL |
| 5 | Filter mantığı | Pro kullanıcı mikrofiber pad vs köpük pad ayrımı ister (microfiber daha agresif kesim) | İHLAL |

**Karar:** **REJECT**
**Alternatif:** `microfiber_pad`'i koru; ileride 2+ ürün geldiğinde ayrı bucket. Tek ürün olsa da filter/semantik bütünlüğü bozmaz.

**Not:** CSV label'ında "8+ ürün gelene kadar geçici merge" yazıyor — ama geçici merge filter'ı bozar, kullanıcı yanlış ürün görür. "Orphan single-product sub" kabul edilebilir.

---

## 4. spare_part — 5 Merge

### 4.1. nozzle_kit → nozzle (81771871)
**81771871:** "IK MULTI 1.5 ve MULTI PRO 2 için Yedek Nozzle Kiti - 2 Parça"
- Mevcut `nozzle` bucket: 1 ürün (83371602 "IK PPF 12 Yedek Tabanca Ucu")

**Rubrik:** K1-5 hepsi OK. nozzle_kit = nozzle (2 parça dahil).
**Karar:** **APPROVE**

### 4.2. trigger_gun → trigger_head (83371816)
**83371816:** "IK PPF 12 Yedek Tabanca"
- Mevcut `trigger_head` bucket: 4 ürün, hepsi "Sprey Başlık / Püskürtücü Başlık"

**Rubrik**
| # | Kriter | Durum |
|---|--------|-------|
| 1 | Primary kategori | Trigger mechanism/spray head | OK |
| 4 | Yapı | 83371816 full tabanca; trigger_head sadece başlık | Belirsiz |

**Karar:** **APPROVE** (marginal) — isim benzerliği semantik olarak aynı aileye işaret ediyor; her ikisi de tetikli püskürtme parçası.
**Not:** Eğer `trigger_head` shopify'da "başlık" filter'ı ise UI'da tabanca-başlık karışabilir. Risk kabul edilebilir — her ikisi de spare_part/spray mechanism.

### 4.3. extension_kit → maintenance_kit (458813)
**458813:** "FLEX EXS M14 **Rotary Polisaj Makinesi Uzatma Aparatı Seti**"
- Mevcut `maintenance_kit` bucket: 3 ürün, hepsi IK basınçlı pompa için yedek köpük/bakım kiti

**Rubrik**
| # | Kriter | Durum |
|---|--------|-------|
| 1 | Primary kategori | Polisaj makinesi uzatması ≠ basınçlı pompa bakım kiti | İHLAL |
| 2 | Hedef ekipman | Polisher vs sprayer pump | İHLAL |
| 3 | Kullanım | Pede erişimi uzatma vs pompa bakım/yenileme | İHLAL |

**Karar:** **REJECT**
**Alternatif:** Yeni sub `polisher_extension` veya mevcut `polisher_accessory`'e (polisher_machine tg altında) taşınmalı. Aslında bu SKU yanlış tg'de — `spare_part` değil `polisher_machine` altında `polisher_accessory` olmalı.

### 4.4. handle → repair_part (82671872)
**82671872:** "IK PRO HANDLE INOX Basınçlı Pompa Püskürtme Kolu"
- Mevcut `repair_part` bucket: 2 ürün, "SGCB Tornador Yedek Boncuk / Kılcal Hortum"

**Rubrik**
| # | Kriter | Durum |
|---|--------|-------|
| 1 | Primary kategori | Yedek parça | OK |
| 2 | Hedef ekipman | IK basınçlı pompa handle; Tornador yedek boncuk | Zayıf benzerlik |
| 3 | Kullanım | Spesifik fonksiyonel parça (kol) vs generic tamir | İHLAL |
| 5 | Filter mantığı | "Kol" arayan kullanıcı repair_part generic bucket'ta kaybolur | İHLAL |

**Karar:** **REJECT**
**Alternatif:** `handle`'ı koru; veya daha uygun sub `pump_handle` / `spray_handle`. Tek ürünlü olsa da anlamı net.

### 4.5. charger → battery (417882) — EN KRİTİK
**417882:** "FLEX 10.8/18V Akü Uyumlu, LED Göstergeli **Şarj Cihazı** (230V - 9A)"
- Mevcut `battery` bucket: 1 ürün (445894 "FLEX AP 18.0V 5.0AH Lithium Power Li-ion **Yedek Akü**")

**Rubrik**
| # | Kriter | Durum |
|---|--------|-------|
| 1 | Primary kategori | Şarj cihazı ≠ batarya. **Fonksiyonel olarak zıt**: biri enerji depolar, diğeri enerji aktarır | KRİTİK İHLAL |
| 4 | Yapı | Charger = AC→DC dönüştürücü devre; battery = Li-ion hücre | İHLAL |
| 5 | Filter mantığı | "Akü arıyorum" diyen kullanıcı şarj cihazı görür — %100 yanlış ürün | KRİTİK İHLAL |

**Karar:** **REJECT (kesin)**
**Alternatif:** `charger` sub_type'ını koru (tek ürünlü de olsa semantik doğru). Veya yeni sub `power_accessory` altında charger + battery + carbon_brush birleştir.

**Rapor'a eklenecek özel uyarı:** Bu merge kararı phase 2 ilk turun yüzeyselliğinin kanıtı. Şarj cihazı ve batarya birleştirmek, sistemin fonksiyonel mantığını bozar.

---

## 5. sprayers_bottles — 1 Merge (REJECT)

### 5.1. dispenser_bottle → pump_sprayer (Q2M-P-DB300M)
**Q2M-P-DB300M:** "GYEON QM Dispenser Bottle **Dağıtıcı Şişe Biberon** - 300 ml"
- `intended_use: Polisaj pastası dozajlama`
- `material: Esnek plastik`
- Uygulama: elle sıkarak damlatma

**Mevcut `pump_sprayer` bucket:** 20+ ürün, hepsi "Basınçlı Pompa" — manuel pompalı, atomized sprey.

**Rubrik**
| # | Kriter | Durum |
|---|--------|-------|
| 1 | Primary kategori | Damla dispenser ≠ basınçlı pompa | İHLAL |
| 3 | Kullanım | Pasta dozajlama (damla) vs kimyasal atomization | İHLAL |
| 4 | Mekanizma | Esnek şişe sıkma vs pompa basıncı | İHLAL |
| 5 | Filter mantığı | "Pompa arıyorum" kullanıcısı 300ml plastik biberon görür | İHLAL |

**Karar:** **REJECT**
**Alternatif:** `dispenser_bottle`'ı koru; veya yeni sub `applicator_bottle` (polisaj pastası uygulama şişeleri için). Phase 3'te applicators tg altına taşınması bile düşünülebilir.

---

## 6. storage_accessories — 2 Merge

### 6.1. bucket_accessories → wash_accessory (79472)
**79472:** "INNOVACAR Araç Yıkama Kovası Seti 20 Lt — Wash Bucket, Grit Guard"

**Rubrik**
| # | Kriter | Durum |
|---|--------|-------|
| 1 | Primary kategori | Araç yıkama aksesuarı | OK |
| 2 | Hedef yüzey | Araç (yıkama süreci) | OK |
| 3 | Kullanım | İki kova yöntemi — yıkama senaryosu | OK |

**Karar:** **APPROVE** — "wash_accessory" yeterince genel, bucket+grit_guard set bu şemsiyede mantıklı.

**Not:** `wash_accessory` şu an storage_accessories altında mevcut sub değil. Yeni bucket açılıyor. Taxonomy'de önceden varsa doğrula; yoksa yeni sub tanımı gerekir.

### 6.2. water_spray_gun → wash_accessory (SGGD402)
**SGGD402:** "SGCB 10 Desenli Su Püskürtme Sulama Tabancası — Bahçe Hortum"

**Rubrik**
| # | Kriter | Durum |
|---|--------|-------|
| 1 | Primary kategori | Su tabancası — yıkama aksesuarı | OK |
| 3 | Kullanım | whenToUse: "Araç yıkama, bahçe sulama" | Kısmen |
| 5 | Filter mantığı | "Sulama tabancası" tek ürün; generic wash_accessory'de kaybolur | ASK |

**Karar:** **ASK** — kriter 5 belirsiz, kendi sub'ında kalırsa daha bulunabilir.

**Soru (Q3):** water_spray_gun tek ürünle ayrı mı kalsın?
- A) Ayrı kalsın (filter için)
- B) wash_accessory altına merge
- **Önerim: A** — "su tabancası" araması spesifik; merge ederse bucket_accessories, eldiven, grit guard ile aynı listede gelir.

---

## 7. tire_care — 3 Merge (Hepsi ASK)

### 7.1-3. tire_gel → tire_dressing (75138, 701908, Q2M-TEYA1000M)

**Kritik veri tutarsızlığı bulundu:**

| SKU | İsim | specs.form | specs.sub_type | Gerçek |
|-----|------|-----------|----------------|--------|
| 75138 | FRA-BER Gommanera Superlux 5lt | **Sıvı (Solvent bazlı)** | `interior_detailer` | Sıvı parlatıcı |
| 701908 | FRA-BER Superlux 900ml | **Jel** | `interior_detailer` | Jel |
| Q2M-TEYA1000M | GYEON Tire Express 1000ml | **Su bazlı jel** | `interior_detailer` | Su bazlı jel |
| 70868 | FRA-BER Gommalux (mevcut tire_dressing) | - | - | Sprey parlatıcı |
| 75140 | FRA-BER Gommanera Superlux 25lt | - | - | Solvent bazlı (75138 ile aynı formül, farklı boyut!) |

**Bulgular:**
1. SKU 75138 adı "tire_gel" ama form = **Sıvı**! Zaten yanlış sınıflanmış.
2. 75138 (5lt) ve 75140 (25lt) aynı ürünün farklı boyutları — biri `tire_gel` diğeri `tire_dressing`. **Variant tutarsızlığı**.
3. specs.sub_type tümünde `interior_detailer` — çok hatalı (lastik ürünü iç bakım değil).

**Rubrik**
| # | Kriter | Durum |
|---|--------|-------|
| 1 | Primary kategori | Lastik parlatıcı/dressing | OK |
| 2 | Hedef yüzey | Lastik yanak | OK |
| 3 | Kullanım | Yıkama sonrası parlatma/koruma | OK |
| 4 | Formülasyon | Form farklı (sıvı/jel); ama tümü dressing | OK |
| 5 | Filter mantığı | "Jel formu" arayan kullanıcı var mı? specs.form ile ayırt edilebilir | Belirsiz |

**Karar:** **ASK** — merge semantik olarak mantıklı ama:
1. 75138'in form'u "Sıvı" — onu tire_dressing'e taşımak zaten doğru (yanlış etiketi düzeltir).
2. 75138 ↔ 75140 variant ilişkisi: ikisi aynı sub'a girmeli.
3. Jel ürünleri (701908, Q2M-TEYA1000M) form=jel ile ayırt edilebilir.

**Soru (Q4):** tire_gel'i tamamen kaldır; specs.form kırılımı ile facet yap?
- A) Tümü tire_dressing'e merge; form specs'te (önerilen, variant ilişkisini de düzeltir)
- B) tire_gel'i koru (701908 + Q2M-TEYA1000M kalsın; 75138 yanlış etiket düzeltilsin → tire_dressing)
- C) Yeni sub_type ayrımı: `tire_dressing_liquid` / `tire_dressing_gel`
- **Önerim (zayıf): A** — variant tutarsızlığını çözer, specs.form filter'ı yeterli.

**Ek not (summary'ye):** specs.sub_type="interior_detailer" tüm 3 üründe hatalı — phase 3 data fix listesine eklenmeli.

---

## Üretilecek Çıktı Dosyaları (onay sonrası)

1. `data/consolidation/phase2R-B-approved.csv`
   - Başlık: `id,scope,sku,field,before,after,label`
   - Satırlar: 11 APPROVE
     - paint_protection_quick: 5 satır (spray_sealant)
     - polisher_machine: 1 satır (516112 → polisher_accessory)
     - spare_part: 2 satır (nozzle, trigger_head)
     - storage_accessories: 1 satır (79472 → wash_accessory)
     - polisher_machine (corded/cordless): Q1 onayı sonrası ekle

2. `data/consolidation/phase2R-B-approved-payload.json`
   - `admin/staging/preview` uyumlu format
   - Her satır: `{scope, sku, field: "template_sub_type", value: <after>}`

3. `data/consolidation/phase2R-B-rejected.md`
   - 11 REJECT (her biri için: SKU, kaynak, hedef, ihlal kriterler, alternatif)
   - En kritik: charger→battery (kesinlikle yapılmamalı)

4. `data/consolidation/phase2R-B-questions.md`
   - 7 soru (Q1-Q7)
   - Q1: corded/cordless rotary merge (power_source filter var mı?)
   - Q2: forced_rotation → da_polisher merge
   - Q3: water_spray_gun ayrı mı kalsın?
   - Q4: tire_gel → tire_dressing (variant tutarsızlığıyla birlikte)
   - Q5-Q7: taxonomy gap — 532579/SGGS003/SGGC086 yanlış tg (heat_gun, air_blower?)

5. `data/consolidation/phase2R-B-summary.md`
   - Sayılar: 11 APPROVE / 11 REJECT / 7 ASK
   - Top 5 bulgu (aşağıdaki)
   - Phase 3'e devredilecekler listesi

---

## En Kritik 5 Bulgu

1. **charger→battery merge'i fonksiyonel mantık ihlali** — şarj cihazı ile bataryanın semantik aksi yönde; phase 2 ilk tur incelemesinin yüzeyselliği bu kararda net görülüyor.

2. **polisher_machine `other` altındaki 5 üründen 4'ü yanlış template_group'ta** — sıcak hava tabancası, hava üfleyiciler ve tornador ayrı tg'lere taşınmalı (heat_gun, air_blower, interior_cleaner). Phase 3 iş kalemi.

3. **polishing_pad merge'leri malzeme/yüzey ihlali** — SGGA081 cam polisaj pedi (wool_pad'e atılamaz, yüzey farklı); NPMW6555 mikrofiber (foam_pad ile malzeme/kesim davranışı farklı). "Geçici merge" felsefesi filter doğruluğuna zarar verir.

4. **tire_care'de variant tutarsızlığı tespit edildi** — 75138 (5lt, tire_gel, form=Sıvı!) ve 75140 (25lt, tire_dressing) aynı ürünün farklı boyutları ama farklı sub_type'ta. Ayrıca tüm 3 üründe specs.sub_type="interior_detailer" hatalı. Phase 3 data quality fix gerekli.

5. **Hedef sub_type'ların mevcut bucket dağılımı kritik** — spray_sealant (5 ürün, genişletilmesi sağlıklı, APPROVE mantıklı) vs wash_accessory (yeni bucket, SGGD402 tek ürün olarak kaybolur riski var, Q3). Merge kararı verilirken hedef bucket semantik homojenliği mutlaka kontrol edilmeli.

---

## Yapılacaklar Listesi (onay sonrası)

- [ ] approved.csv yaz (11 satır; corded/cordless Q1 yanıtına göre 15'e çıkabilir)
- [ ] approved-payload.json üret (jq veya manuel)
- [ ] rejected.md yaz (11 REJECT)
- [ ] questions.md yaz (7 soru, insana sor formatı)
- [ ] summary.md yaz (sayılar + 5 bulgu + phase 3 devir)
- [ ] `admin/staging/preview` ile doğrula (curl POST)
- [ ] Raporda `{total, planned, unsupported, skipped}` skorları göster
- [ ] COMMIT YAPMA — komut açık

---

## Açık Risk/Varsayımlar

- **A1:** `admin/staging/preview` endpoint, `template_sub_type` alanını kabul eder (phase 2'deki payload örneklerinde görüldü).
- **A2:** `wash_accessory` taxonomy'de yeni sub; preview'de "unsupported" gelebilir — varsa alternatif sub seçilmeli (örn. existing `bucket_accessories` koruma).
- **R1:** polisher_machine `other`daki 4 yanlış-tg ürünü REJECT ediyoruz ama phase 3'e kadar yanlış bucket'ta kalacaklar. Acil tg-fix payload'ı ayrı bir faz işi.
- **R2:** Kullanıcıdan 7 soruya yanıt alınmadan approved.csv yalnızca net olan 11 satırla yazılır; sonraki iterasyonda ASK yanıtlarıyla genişletilir.

---

**Plan tamam. Onayla, uygula — yoksa sorular için revize et.**
