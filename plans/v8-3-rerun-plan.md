# v8.3 — Fail Eval Rerun Plan (compact sonrası çalıştırılacak)

## Amaç

v8.2 context retention fix sonrası değişen bot davranışının **39 fail eval** üzerindeki etkisini ölçmek. Hangileri düzeldi, hangileri hala fail, yeni sorun çıktı mı?

## v8.2 Değişikliği Özeti (yapıldı)

Tek dosya değişikliği: `src/conversations/index.ts`
- State schema'ya `lastProducts`, `lastFocusSku`, `lastFaqAnswer` eklendi
- `onAfterTool` hook ile tool sonrası state güncelleniyor
- Instruction'da "HER ZAMAN tool çağır" kuralı → context-aware kural ile değiştirildi (state referanslı)

## Rerun Kapsamı: 39 Fail Eval

**A. Context kritik — 3 eval** (v8.2'nin asıl hedefi):
- 21-bathe-compare-context (son rerun: davranış düzeldi, tool disiplin sürüyor)
- 22-price-sum-context (son rerun: TAM PASSED)
- 23-product-detail-context (son rerun: framework timeout)

**B. No-response / timeout — 11 eval** (idleTimeout sorunu, bot gerçekte cevap veriyor):
02, 24, 25, 26, 27, 31, 33, 39, 52, 53, 56

**C. Diğer fail — 28 eval** (çoğu assertion design, bazıları gerçek):
01, 03, 06, 07, 08, 09, 12, 14, 17, 20, 22, 32, 34, 35, 36, 37, 40, 42, 43, 44, 46, 47, 48, 51, 54, 55
(23 ve 21 A'da listelendi)

## Ek Fix (opsiyonel — timeout için)

Gerçek no-response değil, idleTimeout false-positive. Eğer eval dosyalarına `options: { idleTimeout: 30000 }` eklersek 11 no-response büyük ihtimalle pass olur.

**Karar:** Önce DÜZELT DEĞİL, önce rerun. Sonra gerçek fail vs false-positive ayırımını net gör.

## Execution

### Adım 1: Traces Temizle
```bash
sqlite3 Botpress/detailagent/.adk/bot/traces/traces.db "DELETE FROM spans; DELETE FROM conversation_users; VACUUM;"
```

### Adım 2: Sadece Fail'leri Rerun
`adk evals` **tek eval ismi alıyor**, 39 ayrı komut gerekiyor. 2 strateji:

**Strateji A: Full rerun (58 eval, ~60dk)**
- Basit: `adk evals --json > /tmp/eval-run3.json`
- Dezavantaj: Passed 19 eval da gereksiz koşar
- Avantaj: Önce/sonra tam karşılaştırma

**Strateji B: Loop script (39 eval, ~45dk)**
```bash
# scripts/rerun-failed.sh
FAILED_EVALS=(
  "01-ph-sampuan" "02-foam-sampuan" "03-leather-care" # ... 39 tanesi
)
for eval_name in "${FAILED_EVALS[@]}"; do
  adk evals "$eval_name" --json > "/tmp/eval-run3/${eval_name}.json"
done
```

**Öneri: Strateji A** — Full rerun ile pass→fail regression'ı da yakalarız. 19 pass eval'ı da sağlam mı teyit.

### Adım 3: Önce/Sonra Karşılaştırma

```python
# Scripts/compare_eval_runs.py
import json
with open('/tmp/eval-report.json') as f: r2 = json.load(f)  # v8.1 Run 2
with open('/tmp/eval-run3.json') as f: r3 = json.load(f)  # v8.2 rerun

for e2 in r2['evals']:
  e3 = next((e for e in r3['evals'] if e['name'] == e2['name']), None)
  if e2['pass'] != e3['pass']:
    status = 'FIXED' if e3['pass'] else 'REGRESSED'
    print(f"{status}: {e2['name']}")
```

### Adım 4: Rapor

| Metrik | Run 2 | Run 3 | Delta |
|---|---|---|---|
| Pass | 19/58 | ? | ? |
| Fail | 39/58 | ? | ? |
| Context 3 (21,22,23) | 0/3 pass | ? | ? |
| No-response (11) | 0/11 pass | ? | ? |
| Tool 86% → ? | | | |

**Fixed:** Run 2'de fail, Run 3'te pass
**Regressed:** Run 2'de pass, Run 3'te fail (v8.2 yeni bug)
**Stable fail:** Hala fail, nedeni analiz edilecek

### Adım 5: Sonraki Fazı Belirle

- **Fixed ≥ 15:** v8.2 başarılı, assertion design + idleTimeout fix sonra Instagram fazı
- **Fixed 5-14:** Kısmi başarı, kalıntı sorunları detay incele
- **Fixed < 5 veya Regressed > 0:** v8.2'yi gözden geçir

## Dikkat Edilmesi Gerekenler

1. **Chat integration bağlantısı stabil olsun** — Run 1'de 44 conn-fail olmuştu, Run 2'de 0. Aynı stabilite bekleniyor.
2. **adk dev canlı** — PID 11419 (Botpress/detailagent için). lsof :3001 ile teyit.
3. **Build clean** — `adk build` zero error (v8.2 sonrası teyit edildi)
4. **Rerun tek seferde koştur** — kesilirse yarı rapor alınır, yeniden başla

## Zaman

- Traces temizle: <5s
- Full run: ~60 dakika (LLM judge dahil)
- Karşılaştırma: ~5 dakika
- **Toplam: ~65-70 dakika**

## Komut Sırası (compact sonrası)

```bash
# 1. Durumu teyit
lsof -i :3001 | grep LISTEN  # adk dev çalışıyor mu
cd "/Users/projectx/Desktop/Claude Code Projects/Products Jsons/Botpress/detailagent"
PATH="$HOME/.bun/bin:$PATH" adk build  # zero error olmalı

# 2. Traces temizle
sqlite3 .adk/bot/traces/traces.db "DELETE FROM spans; DELETE FROM conversation_users; VACUUM;"

# 3. Full rerun (background)
rm -f /tmp/eval-run3.json /tmp/eval-run3-stderr.log
PATH="$HOME/.bun/bin:$PATH" adk evals --json > /tmp/eval-run3.json 2> /tmp/eval-run3-stderr.log &
EVAL_PID=$!
echo "Run 3 PID: $EVAL_PID"

# 4. Bekle (~60dk), sonra karşılaştır
python3 Scripts/compare_eval_runs.py /tmp/eval-report.json /tmp/eval-run3.json
```
