# Instagram Konuşmalarını Chatbot Eğitim Formatına Dönüştürme (v2)

## Context

`insta_act/` altında 3 marka Instagram DM dışa aktarımı var (gyeon, menzerna, mgpolish). Toplam **~4.627 konuşma**. Hedef: chatbot eğitimi için minimal format — **müşteri mesajı → markanın verdiği cevap** sırası + hangi marka. Metadata (timestamp, share linkleri, reactions, handle, title) atılır. Kullanıcı isteği netleşti: **cevaplanmış thread'lerde hiçbir mesaj kırpılmayacak**; cevapsız veya sadece-media thread'ler silinmeyecek, ayrı dosyalara kaydedilecek.

---

## Shop / Customer Ayrımı (nasıl belirleniyor?)

`message_1.json` şeması çok net — sample (`gyeon_insta_Act/.../_1453_2027733238622613/message_1.json`):

```json
{
  "participants": [
    { "name": "<customer_name>" },
    { "name": "GYEON Türkiye" }
  ],
  "messages": [ { "sender_name": "...", "content": "...", ... } ],
  "title": "<customer_name>"
}
```

- `participants` **her zaman 2 kişi** içeriyor: biri marka, diğeri müşteri.
- **Marka adı her thread'de sabit** (örn. tüm gyeon thread'lerinde "GYEON Türkiye" geçer).
- **Her brand için marka adı otomatik tespit** edilir: thread'lerin %95+'ında tekrar eden `participants[].name` → shop. Üç marka için beklenen adlar konsola yazdırılıp kullanıcıya gösterilir.
- Her mesajın `sender_name`'i shop adına eşitse `role: "shop"`, değilse `role: "customer"`.

---

## Kaynak Özeti

| Marka     | Inbox | Requests | Toplam |
|-----------|------:|---------:|-------:|
| Gyeon     | 1.798 |       51 |  1.849 |
| Menzerna  | 1.929 |       95 |  2.024 |
| MG Polish |   734 |       20 |    754 |

**Kritik:** tüm metin **mojibake** (UTF-8→Latin-1). `.encode('latin-1').decode('utf-8')` ile düzeltilir. Mesaj dizisi ters kronolojik → `timestamp_ms` artan sırada yeniden sıralanır.

---

## Çıktı Yapısı

Konum: `output/instagram/`

```
output/instagram/
├── conversations.jsonl   # shop en az bir cevap vermiş thread'ler (asıl eğitim verisi)
├── unanswered.jsonl      # sadece customer mesajı var, shop hiç cevap vermemiş
├── media_only.jsonl      # hiçbir mesajda text content yok (sadece share/media)
└── stats.json            # marka × kategori sayıları
```

### `conversations.jsonl` — satır şeması
```json
{
  "brand": "gyeon",
  "is_request": false,
  "turns": [
    {"role": "customer", "text": "..."},
    {"role": "shop", "text": "..."},
    {"role": "customer", "text": "ok"},
    {"role": "shop", "text": "rica ederim"}
  ]
}
```

**Kurallar:**
- Shop en az 1 kez cevap vermişse thread buraya gider
- **HİÇBİR MESAJ KIRPILMAZ** — kısa, "tamam", "ok", "teşekkürler" dahil tüm mesajlar olduğu gibi tutulur
- Mesajlar kronolojik sırada (eskiden yeniye)
- `content` alanı olmayan mesajlar (media-only, unsent, reaction) atlanır — ama thread atılmaz
- Ardışık aynı-role mesajları birleştirilmez, her mesaj ayrı turn

### `unanswered.jsonl` — satır şeması
```json
{
  "brand": "gyeon",
  "is_request": false,
  "customer_messages": ["...", "..."]
}
```
Shop hiç cevap vermemiş, sadece customer mesajı olan thread'ler.

### `media_only.jsonl` — satır şeması
```json
{
  "brand": "gyeon",
  "is_request": false,
  "message_count": 3
}
```
Tüm mesajlarda `content` yok (sadece share/media/reaction). Metin bilgisi olmadığı için chatbot için kullanışsız ama sayım için tutuluyor.

### `stats.json`
```json
{
  "gyeon":    { "conversations": N, "unanswered": N, "media_only": N, "total_turns": N },
  "menzerna": { ... },
  "mgpolish": { ... },
  "shop_names_detected": {"gyeon":"GYEON Türkiye", "menzerna":"...", "mgpolish":"..."}
}
```

---

## Uygulama Planı

### Adım 1: `scripts/insta_merge/decode_utils.py`
- `fix_mojibake(s) -> str`: `s.encode('latin-1').decode('utf-8')`, exception olursa `s` döner

### Adım 2: `scripts/insta_merge/detect_shop.py`
- Her marka için tüm `participants[].name` değerlerini sıklığa göre say
- Thread sayısının %80+'ında görünen tek sabit ad → shop adı
- Üç marka için tespit edilen adları konsola yazdır (kullanıcı doğrulayabilsin)
- Döner: `{"gyeon": "...", "menzerna": "...", "mgpolish": "..."}`

### Adım 3: `scripts/insta_merge/build_jsonl.py`
Her marka için her `message_1.json`:
1. JSON yükle, `fix_mojibake` uygula (tüm string alanlara recursive)
2. `messages[]`'i `timestamp_ms` artan sırala
3. Thread'i sınıflandır:
   - En az 1 shop + 1 customer `content`'li mesaj var → **conversations.jsonl**
   - Yalnızca customer `content`'li mesajı var → **unanswered.jsonl**
   - Hiçbir mesajda `content` yok → **media_only.jsonl**
4. İlgili dosyaya append et
5. `is_request` = thread'in `message_requests/` altından gelip gelmediği

### Adım 4: `stats.json` üret

### Adım 5: `scripts/insta_merge/run_all.py` — orchestrator

### Adım 6: Doğrulama
- `wc -l` ile 3 çıktı dosyasının satır sayısı toplamı ≈ 4.627
- Rastgele 10 `conversations.jsonl` satırı göz kontrol: shop/customer rolleri mantıklı mı, Türkçe düzgün mü
- `grep -c 'Ã' output/instagram/conversations.jsonl` → 0 (mojibake sıfır)
- Shop adları konsolda: "GYEON Türkiye", "Menzerna Türkiye (veya benzeri)", "MG Polishing (veya benzeri)"

---

## Kritik Dosyalar

Oluşturulacak:
- `scripts/insta_merge/decode_utils.py`
- `scripts/insta_merge/detect_shop.py`
- `scripts/insta_merge/build_jsonl.py`
- `scripts/insta_merge/run_all.py`

Çıktılar:
- `output/instagram/conversations.jsonl`
- `output/instagram/unanswered.jsonl`
- `output/instagram/media_only.jsonl`
- `output/instagram/stats.json`

---

## Sonraya Bırakılan

- **Handle / PII**: Mevcut çıktı formatında handle/title zaten yok. İleride müşteri bazlı analiz gerekirse hash'li `customer_id` eklenebilir — kullanıcıya implementasyon sonrası tekrar sorulacak.
