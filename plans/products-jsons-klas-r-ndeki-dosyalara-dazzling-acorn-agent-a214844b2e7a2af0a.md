# Phase 2 Revize — Küçük Grup Merge + TG Merge + Orphan Fix İnceleme Planı

**Durum:** Plan mode — sadece inceleme tamamlandı, karar planı hazır. Kullanıcı onayından sonra 5 çıktı dosyası üretilecek.

**Kapsam:** `phase2R-C-*` üretimleri: A) küçük grup sub_type merge, B) template_group merge (8 SKU), C) orphan fix (6 SKU), D) accessory orphan grup eritme.

---

## Canlı Veri (microservice) — Özet Kanıt

### A1 — leather_care
| SKU | Name | Current Sub | Önerilen | Fonksiyon |
|---|---|---|---|---|
| 700468 | FRA-BER Cream Leather | `leather_conditioner` | `leather_protectant` (merge) | Besleme+Nemlendirme (koruyucu film var ama temel: conditioner) |
| Q2-LCR500M | GYEON Q Leather Coat | `leather_protectant` | korundu | SiO2 hidrofobik kaplama, 3 ay kalıcı |

Taxonomy: leather_care (7) — leather_cleaner=3, leather_care_kit=2, **leather_conditioner=1, leather_protectant=1**.

### A2 — brushes (SGGD294)
- Name: "SGCB **Tire** Cleaning Brush Lastik Davlumbaz"
- Current sub: `wheel_brush` — **yanlış etiket**
- targetSurface: Lastik, zemin paspası, halı
- Önerilen: `tire_brush` — doğru.

### A3 — glass_cleaner_protectant (Q2M-GPYA1000M)
- Name: "GYEON QM Glass+ Plus Hidrofobik Parlak"
- Current sub: `glass_hydrophobic_sealant`
- function: "Temizlik + Hidrofobik Koruma"
- Önerilen: `glass_protectant` — isim sadeleştirme (taxonomy'de glass_protectant yok, yaratılacak)

### B — Template Group merge (8 SKU)

| SKU | Name | tg before | tg after | Notu |
|---|---|---|---|---|
| 701606 | FRA-BER LAVAVETRO Cam Suyu Katkısı | glass_cleaner | glass_care | **WASHER FLUID ADDITIVE** — araç cam suyu deposu |
| 74955 | FRA-BER Toglimoscerini Cam Suyu Katkısı | glass_cleaner | glass_care | **WASHER FLUID ADDITIVE** |
| 700466 | FRA-BER Lindo Cam/Lake Temizleyici | glass_cleaner_protectant | glass_care | Sprey-sil cam temizleyici |
| 71176 | FRA-BER Reflex Cam/Lake/Laminant Temizleyici | glass_cleaner_protectant | glass_care | Sprey-sil |
| 700662 | INNOVACAR 0 AMMONIA Cam Temizleyici | glass_cleaner_protectant | glass_care | Sprey-sil |
| Q2M-GPYA1000M | GYEON QM Glass+ Plus | glass_cleaner_protectant | glass_care | Temizlik+hidrofobik koruma |
| JC0101 | Little Joe Ekran Temizleme | glass_cleaner_protectant | glass_care | Telefon/tablet/LED |
| **Q2M-PYA4000M** | **GYEON QM Prep** (IPA panel wipe) | contaminant_solvers | **ceramic_coating** | Bağımsız vaka — kaplama öncesi yüzey hazırlayıcı |

### C — Orphan fix (6 SKU, 5 polisher_machine + 1 accessory)

| SKU | Name | Current | Önerilen sub | Notu |
|---|---|---|---|---|
| 26942.099.001 | MENZERNA Mikrofiber Bez Seti | accessory / NULL | microfiber_cloth | mainCat=AKSESUAR, subCat=Yardımcı Ürünler |
| 516112 | FLEX FS 140 Uzatma Aparatı | polisher_machine/other | extension_shaft | subCat2=Uzatma Aparatları ✓ |
| 532579 | FLEX HG 650 Sıcak Hava Tabancası | polisher_machine/other | heat_gun | subCat=Hava Tabancaları ✓ |
| SGGC055 | SGCB Tornador | polisher_machine/other | tornador_gun | subCat=Tornadorlar ✓ |
| SGGC086 | SGCB Air Blow Gun | polisher_machine/other | air_blow_gun | subCat=Hava Tabancaları ✓ |
| SGGS003 | SCCB Kısa Nozul Hava Tabancası | polisher_machine/other | air_blow_gun | subCat=Hava Tabancaları ✓ |

### polisher_machine taxonomy — tüm sub'lar
`corded_rotary_polisher=3, mini_cordless_polisher=5, other=5, da_polisher=4, forced_rotation_polisher=2, sander=2, cordless_rotary_polisher=1, machine_kit=1`. Grup zaten **machine_equipment** gibi davranıyor (sander, machine_kit, mini_cordless_polisher hepsi farklı ekipman kategorisi). heat_gun / tornador_gun / air_blow_gun / extension_shaft sub'ları bu şemsiye grupta **tutarlı** (eşdeğer sub_type genişlemesi). Ancak grup **ADI yanıltıcı** → gelecek phase için rename (machine_equipment / power_tools) önerilecek.

### accessory grubu (tam eritme fırsatı)
- Grup toplam: **1 SKU** (26942.099.001)
- Diğer tüm mikrofiber: `microfiber` grubu (31 SKU, microfiber_cloth yok ama `multi_purpose_cloth`, `buffing_cloth`, `kit` var)
- Orphan-fix payload sadece sub_type=microfiber_cloth atıyor, **template_group=accessory olarak bırakıyor**. Bu yanlış: 1-ürünlük grubu eritmek gerek.

---

## Karar Matrisi

### APPROVE (6 değişiklik, tam net)

1. **phase2c-brushes-1** (SGGD294 wheel_brush→tire_brush) — ürün adı+targetSurface tam uyumlu. %99.
2. **phase2c-glassp-2** (Q2M-GPYA1000M glass_hydrophobic_sealant→glass_protectant) — simetri iyileşmesi. %92.
3. **phase2c-orphan-2** (516112 → extension_shaft) — subCat2 canonical. %99.
4. **phase2c-orphan-3** (532579 → heat_gun) — specs.product_type canonical. %99.
5. **phase2c-orphan-4** (SGGC055 → tornador_gun) — subCat=Tornadorlar. %98.
6. **phase2c-orphan-5/6** (SGGC086, SGGS003 → air_blow_gun) — specs.product_type canonical. %99.

### ASK (3 belirsiz karar, kullanıcı onayı gerek)

**Q1: leather_conditioner → leather_protectant merge (700468 Cream Leather)**
- Conditioner ≠ protectant fonksiyon olarak (besleme vs. SiO2 bariyer).
- Merge sonrası bot "deri nemlendirici" aramasında SiO2 kaplama da dönebilir → relevance drop.
- Alternatifler: (A) Merge'i iptal et, ayrı kalsın. (B) Merge'i yap ama label "leather_protectant_or_conditioner" gibi daha kapsayıcı. (C) Reverse mapping ekle (bot soru parametresine göre filtrele).
- **Önerim:** A (reject merge) — semantik kayıp çok yüksek.

**Q2: glass_cleaner + glass_cleaner_protectant → glass_care merge (7 SKU)**
- İki sub-kategori kritik farklı: cam suyu katkısı (701606, 74955) deposu ürünleri VS sprey-sil cam temizleyici (700466, 71176, 700662, Q2M-GPYA1000M). Ekran temizleyici (JC0101) de farklı use-case.
- "Cam temizleyici istiyorum" sorusu katkı döndürürse yanlış.
- Ama sub_type ayrımı korunuyor (glass_cleaner_additive, glass_cleaner, glass_hydrophobic_sealant/glass_protectant, screen_cleaner) → bot filter mantığı sub'a göre hala ayırabilir.
- Alternatifler: (A) Merge yap (tek grup, 4 sub). (B) glass_cleaner ayrı kalsın (windscreen washer additive'leri için ayrı taxonomy). (C) 3 grup: glass_washer_additive, glass_cleaner, screen_cleaner.
- **Önerim:** A (approve) **eğer bot filter mantığı sub_type priority verirse** — aksi halde B.
- **Pre-merge şartı:** staging.ts `template_group` whitelist'e eklendi mi? `phase2-tgmerge-glass-need-staging-update.md` dosyası bu engelden bahsediyor. **Bu blocker çözülmeden preview test bile yapılamaz.**

**Q3: Q2M-PYA4000M contaminant_solvers → ceramic_coating**
- IPA prep panel wipe, kaplama öncesi yüzey hazırlayıcı.
- contaminant_solvers'ta "single_layer_coating" sub_type olması mantıksız (zaten orada tek bu ürün var).
- ceramic_coating'de `single_layer_coating` var (Q One EVO=9H kaplama, Q Primer=prep+son-kat). Q2M-PYA4000M Q Primer'e benzer (prep odaklı).
- Ancak Prep panel wipe ana "kaplama" değil, prep. Temiz çözüm: yeni `surface_prep` sub_type.
- Alternatifler: (A) Taşı→ceramic_coating, sub=single_layer_coating (önerilen payload). (B) Taşı→ceramic_coating, yeni sub=surface_prep. (C) contaminant_solvers'ta kal, sub=oil_degreaser (IPA yağ sökücü).
- **Önerim:** B — ceramic_coating hedef doğru, ama `single_layer_coating` yanlış — `surface_prep` sub yaratılmalı.

### ÖZEL DURUM (D) — accessory grup eritme

**Q4: 26942.099.001 sub_type=microfiber_cloth + template_group=accessory→microfiber**
- Payload sadece sub_type atıyor. Ama grup 1-ürünlük.
- microfiber grubunda `microfiber_cloth` sub'u yok (var olanlar: kit, multi_purpose_cloth, buffing_cloth, glass_cloth, wash_mitt, drying_towel, interior_cloth, coating_cloth, suede_cloth, chamois_drying_towel, cleaning_cloth, interior_cleaning_applicator).
- MENZERNA Mikrofiber Bez Seti 4'lü paket = genel amaçlı silme bezi = `multi_purpose_cloth` en iyi eşleşme.
- Alternatifler: (A) Sadece sub=microfiber_cloth (mevcut payload). (B) tg=microfiber + sub=multi_purpose_cloth. (C) tg=microfiber + sub=kit (4'lü paket).
- **Önerim:** B — mevcut taxonomy ile simetri, grubu da eritir.

### POLISHER_MACHINE NOTU (ileride rename)

Grup adı yanıltıcı. Önerilen (bu phase'te DEĞİL): `polisher_machine → machine_equipment` veya `power_tools`. Heat gun, tornador, air blow gun, extension shaft, sander, machine_kit sub'ları "polisher" çerçevesine uymuyor ama makine-ekipman üst kümesine uyuyor.

### REJECT (yok, hiçbir payload change primary-kategori ihlali içermiyor)

---

## Blocker'lar

**B1:** staging.ts whitelist `template_group` desteklemiyor. B vakası (glass_care merge + Q2M-PYA4000M tg move) preview bile edilemez. Kod değişikliği gerek (`retrieval-service/src/routes/admin/staging.ts:62`).

**B2:** Yeni sub_type'lar (tire_brush, glass_protectant, extension_shaft, heat_gun, tornador_gun, air_blow_gun, surface_prep önerisi, microfiber_cloth) mevcut taxonomy'de yok → taxonomy insert/allowlist update gerek olabilir. Staging endpoint yeni sub_type'ları kabul eder mi kontrol edilmeli.

---

## Çıktı Dosyası Şeması (onay sonrası üretilecek)

### `phase2R-C-approved.csv`
```
id,scope,sku,field,before,after,label
phase2R-C-brushes-1,product,SGGD294,template_sub_type,wheel_brush,tire_brush,P0 APPROVE
phase2R-C-glassp-1,product,Q2M-GPYA1000M,template_sub_type,glass_hydrophobic_sealant,glass_protectant,P1 APPROVE
phase2R-C-orphan-2,product,516112,template_sub_type,other,extension_shaft,P0 APPROVE
phase2R-C-orphan-3,product,532579,template_sub_type,other,heat_gun,P0 APPROVE
phase2R-C-orphan-4,product,SGGC055,template_sub_type,other,tornador_gun,P0 APPROVE
phase2R-C-orphan-5,product,SGGC086,template_sub_type,other,air_blow_gun,P0 APPROVE
phase2R-C-orphan-6,product,SGGS003,template_sub_type,other,air_blow_gun,P0 APPROVE
```
(7 satır. B=hepsi ASK, Q1 ASK, Q4 ASK — onay gelirse eklenir.)

### `phase2R-C-approved-payload.json`
Yukarıdaki 7 satırın changes[] dizisi.

### `phase2R-C-rejected.md`
- Hiçbir REJECT yok. "Tümü APPROVE veya ASK" notu.

### `phase2R-C-questions.md`
Q1 (leather merge), Q2 (glass_care merge), Q3 (IPA prep hedef), Q4 (accessory grup eritme) + şema örnek payload'ları.

### `phase2R-C-summary.md`
- Durum: 7 APPROVE / 4 ASK / 0 REJECT
- Blocker'lar (staging.ts whitelist, taxonomy sub_type allowlist)
- Polisher_machine rename önerisi (out-of-scope, next phase)
- Preview beklenen çıktı: `total=7, planned=7, unsupported=0, skipped=0` (sadece APPROVE'lar için — hiçbiri template_group değiştirmiyor)

---

## Sonraki Adımlar (kullanıcı planı onaylarsa)

1. `phase2R-C-approved.csv` + payload.json üret (7 APPROVE)
2. `phase2R-C-rejected.md` (boş/not)
3. `phase2R-C-questions.md` (4 Q)
4. `phase2R-C-summary.md`
5. Preview doğrulama: `staging/preview` endpoint'e payload gönder, `planned=7` beklentisi
6. Commit YAPMA (talep dışı).

**Tahmini süre:** 10-15 dk (dosya üretimi + preview doğrulama).

**Yalnızca plan mode — edit yok.**
