# Shopify Store Entegrasyonu — Faz 2: Tema + Admin + İçerik

## Context

Faz 1'de Shopify Dev MCP kuruldu ve çalışıyor (dokümantasyon/şema/validation için). Faz 2'de **gerçekten mağazaya yazma yetisi** ekliyoruz. Kullanıcının hedefleri:

1. **Tema geliştirme** — Krank temasını (satın alınacak, şu an demo) local'de düzenleme, preview, push
2. **Ürün/koleksiyon/envanter yönetimi** — ürün CRUD, metafield, stok
3. **İçerik yönetimi** — blog, sayfa, SEO, metafield

Kullanıcı **store owner** — custom app oluşturup Admin API access token alabilir.

### Araç Seçimleri (araştırma sonucu)

| İhtiyaç | Araç | Neden |
|---|---|---|
| Tema dev (Liquid) | **Shopify CLI** (`@shopify/cli`) | Tek resmi yöntem; `theme pull/dev/push` + browser OAuth |
| Ürün, müşteri, sipariş, metafield, envanter, tag | **GeLi2001/shopify-mcp** MCP | 192★, aktif, GraphQL Admin API 2026-01, mutation destekli, 31 tool |
| Blog, article, page | **Admin API + Bash/curl** (Dev MCP schema desteğiyle) | Community MCP'ler ya stale (luckyfarnon 5★, Jan 2025) ya eksik. Dev MCP zaten şemayı sağladığı için Claude GraphQL'i yazar, curl çalıştırır — stale bir MCP'ye bağımlılık yok |

**Neden GeLi2001, başkası değil?**
- `antoineschaller/shopify-mcp-server`: 14★, metafield yok
- `luckyfarnon/Shopify-MCP`: 5★, son commit Ocak 2025 (stale), npm package adı çakışması riski
- `GeLi2001/shopify-mcp`: 192★, son güncelleme yakın, metafield + tag + inventory var, npm'de `shopify-mcp` paketinin aktif sahibi

## Adım 1 — Shopify Admin API Access Token Oluştur (kullanıcı yapar)

**Claude otomatikleştiremez — kullanıcı manuel yapmalı:**

1. Shopify admin → **Settings → Apps and sales channels → Develop apps**
2. "Allow custom app development" etkinleştir (ilk kez yapılıyorsa)
3. **Create an app** → isim: `claude-code-integration`
4. **Configure Admin API scopes** — aşağıdaki scope'ları işaretle:
   - `read_products`, `write_products`
   - `read_product_listings`
   - `read_inventory`, `write_inventory`
   - `read_content`, `write_content` *(blog, article, page)*
   - `read_themes`, `write_themes` *(CLI ile push için)*
   - `read_customers`, `write_customers` *(opsiyonel)*
   - `read_orders`, `write_orders` *(opsiyonel)*
   - `read_metaobjects`, `write_metaobjects`
5. **Install app** → Admin API access token gösterilir (sadece BİR KEZ!)
6. Token'ı güvenli yerde sakla; mağaza domain'ini not et (ör. `krank-demo.myshopify.com`)

**Güvenlik:** Token custom app'e bağlı; istediğinde Develop Apps'den revoke edebilirsin. Repo'ya commit etme.

## Adım 2 — Shopify CLI Kur (tema için)

```bash
npm install -g @shopify/cli@latest
shopify version   # doğrulama
```

**Alternatif (macOS):** `brew tap shopify/shopify && brew install shopify-cli`

İlk tema komutu (aşağıdaki 3. adımda) çalıştırıldığında browser açılır, mağaza sahibi olarak Shopify'a giriş yapılır — OAuth flow token'ı yerelde saklar. Ayrı config gerekmez.

## Adım 3 — Tema Geliştirme Workspace'i Hazırla

Henüz Krank satın alınmamış; demo ile çalışılacak. İki senaryo:

**Senaryo A: Demo/mevcut bir tema ile başla**
```bash
mkdir -p ~/Desktop/shopify-mcp/krank-theme
cd ~/Desktop/shopify-mcp/krank-theme
shopify theme pull --store krank-demo.myshopify.com
# tema seçim listesi gelir → mevcut aktif/demo tema seçilir
```

**Senaryo B: Krank satın alındıktan sonra**
Satın alma → Online Store → Themes'te tema belirir → `shopify theme pull` ile local'e çek.

**Local preview:**
```bash
shopify theme dev --store krank-demo.myshopify.com
# http://127.0.0.1:9292 açılır, hot-reload aktif
```

**Push (prod'a yükleme — dikkat!):**
```bash
shopify theme push --unpublished       # yeni unpublished tema olarak
# VEYA
shopify theme push --theme <id>        # belirli bir temanın üzerine
```
Canlı tema üzerine push ASLA otomatik yapılmayacak — sadece kullanıcı onayıyla.

## Adım 4 — Admin MCP'yi Kur (ürün/müşteri/sipariş/metafield/envanter için)

```bash
claude mcp add shopify-admin \
  --scope user \
  --env SHOPIFY_ACCESS_TOKEN=shpat_XXXXXXXXXXXXXX \
  --env MYSHOPIFY_DOMAIN=krank-demo.myshopify.com \
  --env SHOPIFY_API_VERSION=2026-01 \
  -- npx -y shopify-mcp@latest
```

**Not:** Adım 1'deki gerçek token ve domain ile değiştirilecek. Token `shpat_` prefix ile başlar.

**Doğrulama:**
```bash
claude mcp list
claude mcp get shopify-admin
# Status: ✓ Connected olmalı
```

Yeni Claude Code oturumunda test:
> "Mağazamda kaç ürün var, ilk 5'ini listele"

`get-products` / `search-products` tool'ları çağrılmalı.

## Adım 5 — Blog/Page/Content için Pattern Dokümantasyonu

GeLi2001 MCP blog/page içermediği için bu operasyonlar **Bash + curl + Dev MCP schema** ile yapılacak. Claude iki tool'u birlikte kullanır:

1. **Dev MCP'nin `introspect_admin_schema`** → `articleCreate`, `pageCreate`, `blogCreate` mutation imzalarını bul
2. **Dev MCP'nin `validate_graphql_codeblocks`** → yazılan query'yi doğrula
3. **Bash** → curl ile çalıştır:

```bash
curl -s -X POST "https://$MYSHOPIFY_DOMAIN/admin/api/2026-01/graphql.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query":"mutation { articleCreate(article: {...}) { article { id title } userErrors { field message } } }"}'
```

**Env export pattern** (yeni shell oturumunda kullanmak için — opsiyonel, sadece bu workflow aktifken):
```bash
# ~/.zshrc'ye EKLEME — KULLANICI ONAYIYLA
export SHOPIFY_ACCESS_TOKEN="shpat_XXX"
export MYSHOPIFY_DOMAIN="krank-demo.myshopify.com"
```
Ya da `.env.shopify` ayrı dosya (gitignore'da):
```bash
source ~/.env.shopify && curl ...
```

Bu adımın çıktısı: plan dosyasına ek olarak mağaza entegrasyon klasöründe [~/Desktop/shopify-mcp/ADMIN_API_PATTERNS.md](~/Desktop/shopify-mcp/ADMIN_API_PATTERNS.md) notu — içerikte blog/page/metafield için örnek curl snippet'leri ve env setup. (Bu dosya sadece kullanıcı için referans, kod değil.)

## Adım 6 — Doğrulama (End-to-End)

1. **Dev MCP** (önceden kurulu): `introspect_admin_schema` ile `productCreate` imzasını bul → çalışmalı
2. **Shopify CLI**: `shopify theme list --store <domain>` → tema listesi dönmeli
3. **Admin MCP**: "mağazadaki son 5 siparişi getir" → tool çağrısı başarılı olmalı
4. **Curl pattern**: Dev MCP ile yazılmış bir `pageCreate` mutation'ını curl ile çağır → `userErrors: []` dönmeli

## Kritik Dosyalar

- [~/.claude.json](~/.claude.json) → `mcpServers.shopify-admin` yeni entry
- [~/Desktop/shopify-mcp/krank-theme/](~/Desktop/shopify-mcp/krank-theme/) → tema workspace (yeni klasör)
- [~/Desktop/shopify-mcp/ADMIN_API_PATTERNS.md](~/Desktop/shopify-mcp/ADMIN_API_PATTERNS.md) → blog/page/metafield curl örnekleri (kullanıcı referansı)
- **Değiştirilmeyecek:** Mevcut `shopify-dev-mcp` (Faz 1) ve `simplemem` MCP entry'leri

## Riskler & Güvenlik

- **Token sızması:** Admin API token `shpat_` prefix'li, mağazanın tam kontrolünü verir. `~/.claude.json` dosyasına gömülecek — bu dosya user-only okuma izinli olmalı (`chmod 600 ~/.claude.json` kontrol edilecek). Repo'ya asla commit edilmeyecek.
- **Minimal scope:** Adım 1'deki scope listesi gerekli olanın üstüne çıkmıyor. Order/customer opsiyonel, istenirse eklenir.
- **Canlı tema push:** `shopify theme push` canlıya yazar. Plan gereği bu komut Claude tarafından ASLA otomatik çalıştırılmayacak, sadece kullanıcı onayı ile. `--unpublished` flag'i default olacak.
- **Topluluk MCP güvenilirliği:** `shopify-mcp` paketi Shopify'ın resmi paketi DEĞİL (GeLi2001 topluluk projesi). Güncelleme gelmezse, alternatif olarak Admin API'yı doğrudan curl ile kullanabiliriz (zaten Adım 5'in yaklaşımı). Dev MCP resmi olduğu için temel fallback her zaman çalışır.
- **Stale credentials:** Token uzun süre kullanılmazsa invalidate edilmez ama `shopify auth logout` ile CLI token'ları temizlenebilir.
- **API version drift:** `2026-01` sabitlendi; Shopify quarterly release yaparken plan güncellenecek.

## Rollback

```bash
claude mcp remove shopify-admin --scope user
npm uninstall -g @shopify/cli
# Admin'den: Develop apps → claude-code-integration → Uninstall → token revoke
```

## Kaynaklar

- [Shopify CLI theme](https://shopify.dev/docs/api/shopify-cli/theme)
- [GeLi2001/shopify-mcp](https://github.com/GeLi2001/shopify-mcp)
- [Shopify Admin API GraphQL](https://shopify.dev/docs/api/admin-graphql)
- [Custom apps in Shopify admin](https://help.shopify.com/en/manual/apps/app-types/custom-apps)
