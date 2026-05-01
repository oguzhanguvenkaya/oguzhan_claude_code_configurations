# Plan: `/ultrareview` Dokümantasyonunu Okuma

## Context

Kullanıcı `https://code.claude.com/docs/en/ultrareview` URL'ini paylaşıp "bunu okur musun" diye sordu. Bu bir **bilgi toplama / okuma** isteği; kod değişikliği veya implementasyon gerektirmiyor. İstek WebFetch ile karşılandı ve içerik özetlendi.

## Doküman Özeti (Temel Noktalar)

- **`/ultrareview`**: Claude Code on the web altyapısında, bulut sandbox'ında çalışan **çoklu-ajan derin code review** aracı.
- Amaç: merge öncesi, bağımsız olarak **reproduce + verify** edilmiş gerçek bug'ları yakalamak.
- **Gereksinimler**: Claude Code v2.1.86+, Claude.ai auth (API key yetmez), extra usage etkin.
- **Kullanılamayan durumlar**: Bedrock, Vertex AI, Foundry, ZDR organizasyonları.
- **Kullanım**:
  - `/ultrareview` → branch vs default branch diff'i review eder.
  - `/ultrareview <PR#>` → GitHub PR'ı clone edip review eder.
- **Süre**: ~5–10 dk, arka planda; `/tasks` ile takip.
- **Ücretlendirme**:
  - Pro/Max: 5 Mayıs 2026'ya kadar 3 ücretsiz run (tek seferlik, yenilenmez).
  - Team/Enterprise: ücretsiz yok.
  - Sonrası: run başına ~$5–$20 extra usage.
- **`/review` ile fark**: `/review` yerel + tek-geçiş (hızlı iterasyon); `/ultrareview` bulut + multi-agent (merge-öncesi derinlik).
- **Yalnızca manuel invoke** ile çalışır; Claude kendiliğinden başlatmaz.

## Önerilen Sonraki Adım

Bu istek salt okuma olduğu için **herhangi bir kod değişikliği yapılmayacak**. Bir sonraki adım kullanıcıya kalmış:

1. Mevcut Phase 4 branch'i (`feat/phase-4-tool-cutover`) üzerinde `/ultrareview` denemek,
2. Açık PR varsa `/ultrareview <PR#>` ile PR modunda çalıştırmak,
3. Ya da sadece bilgi olarak not etmek ve başka işe geçmek.

## Verification

N/A — bilgi okuma isteği, kod değişikliği yok.
