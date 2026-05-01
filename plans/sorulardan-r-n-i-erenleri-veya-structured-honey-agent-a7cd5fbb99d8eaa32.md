# Plan: Translate Gyeon FAQ Chunk 6 (109 rows) to Turkish

## Status

Plan mode is active per system-reminder, which supersedes the user's "EXECUTE NOW" directive. I cannot write the Python script, run it, or produce the CSV until the user exits plan mode and approves.

## Inputs verified

- Source CSV: `/Users/projectx/Desktop/Claude Code Projects/Products Jsons/botpress/detailagent/output/staging/chunks/chunk_6.csv`
- Row count: 109 logical rows confirmed (lines 2-110 of the CSV).
- Output path required: `/Users/projectx/Desktop/Claude Code Projects/Products Jsons/botpress/detailagent/output/staging/translations/chunk_6_tr.csv`
- Output dir does not exist yet — script must create it.

## Output schema

Columns: `target_sku, section_name, category_name, question_tr, answer_tr, question_en, answer_en, url, article_id`

## Translation approach

A single Python script `/tmp/translate_chunk6.py` will:
1. Read source CSV with `csv.DictReader`.
2. Look up each row's `article_id` in an inline dict `TRANSLATIONS = { article_id_str: {"question_tr": "...", "answer_tr": "..."} }` containing all 109 entries.
3. Build output rows preserving `target_sku, section_name, category_name, url, article_id` verbatim and appending `question_tr`, `answer_tr`, `question_en`, `answer_en`.
4. Write with `csv.DictWriter`, `quoting=csv.QUOTE_MINIMAL`, UTF-8 (no BOM), LF newlines.
5. Assert all 109 article_ids resolved (no missing translations) and print row count + samples.

## Translation rules to apply

- Product names verbatim with `Q²` superscript: `Q²M Bathe+`, `Q²M Compound`, `Q²M Iron REDEFINED`, `Q²R FabricCoat`, `Q² LeatherCoat REDEFINED`, etc.
- Units verbatim: `9H`, `pH 2-11`, `50 ml`, `25.000 km` (comma → dot for TR thousands), `°C`, `IR lamp`, `SiO²`, `µm`, `30°C` (the source uses `30’C` typo — translate to `30°C`).
- `24 months` → `24 ay`; `3 years` → `3 yıl`; `15 minutes` → `15 dakika`; `12 hours` → `12 saat`; `3 months` → `3 ay`.
- URLs unchanged. `Gyeon`, `IPA`, `Alcantara`, `UV` unchanged.
- "siz" form, natural Turkish, detailing terminology (kaplama, cila, mikrofiber, yıkama, parlatma, kontaminasyon, demir parçacıkları, katran, hidrofobik, kuvars bazlı).

## Repeated phrases (template translations)

The most common repeated answers — translate consistently:

- EN: "Secure the dispenser in the closed position. Store it in a dry, dark, and cool place. Please do not allow it to freeze or expose it to sunlight. Once opened, it has a 'good-to-use' shelf life of 3 years."
  TR: "Dağıtıcıyı kapalı konumda sabitleyin. Kuru, karanlık ve serin bir yerde saklayın. Lütfen donmasına izin vermeyin ve güneş ışığına maruz bırakmayın. Açıldıktan sonra 3 yıl 'kullanıma uygun' raf ömrüne sahiptir."
  (Variant for `Q²M ClayBars` row uses original-container/15-30°C wording; one row says "three years" — keep "üç yıl".)

- EN: "How to store the product after use? What is the shelf life once opened?"
  TR: "Ürün kullanım sonrası nasıl saklanmalıdır? Açıldıktan sonra raf ömrü nedir?"

- EN: "How to wash/maintain the product?" / EN answer: "It can be Machine washed, at maximum 30'C with added Q²M TowelWash as detergent. Do not tumble dry. Do not iron. Do not bleach."
  TR Q: "Ürün nasıl yıkanır/bakımı yapılır?"
  TR A: "Deterjan olarak Q²M TowelWash eklenerek en fazla 30°C'de makinede yıkanabilir. Kurutma makinesinde kurutmayın. Ütülemeyin. Çamaşır suyu kullanmayın."

- EN: "How to use it?" — TR: "Nasıl kullanılır?"
- EN: "What is it?" — TR: "Bu ürün nedir?"
- EN: "Can I top it off with additional coating?" — TR: "Üzerine ek bir kaplama uygulayabilir miyim?"
- EN: "Will [X] damage/alter my paint work or coating?" — TR: "[X] boyamı veya kaplamamı zedeler/değiştirir mi?"
- EN: "Some contamination will require multiple applications for optimum result(s)." — TR: "Bazı kontaminasyonlar için en iyi sonucu almak adına birden fazla uygulama gerekebilir."
- EN: "I put some compound on a plastic part - How to remove it?" / "Wipe multiple times with Q²M Prep" — TR: "Plastik bir parçaya pasta sürdüm — nasıl temizlerim?" / "Q²M Prep ile birkaç kez silin."
- EN: "Overusing the product will result in high spot - Proceed with washing the panel concerned and re-apply." — TR: "Ürünün fazla kullanımı leke (high spot) bırakır — ilgili paneli yıkayıp yeniden uygulayın."

## Per-row translations (109 entries, keyed by article_id)

The script will contain a dict with these 109 entries. Construction is mechanical given the templates above plus row-specific Q/A translations for unique rows (e.g. 360000981377 polishing residue, 360000962177 Prep alcohol composition, 360000945838 LeatherCleaner Natural pH7-8, 360000945518 LeatherCleaner Strong pH10, 360000942858 Tire Express 15 minutes, 360001038817 FabricCoat sticky surface, 360001047958 FabricCoat application, 360001047918 FabricCoat product description, 360001047258 VinylCleaner description, etc.).

Estimated unique-Q/A rows: ~45. Repeated-template rows: ~64 (mostly the storage shelf-life template and microfiber wash template). All will be authored explicitly — no placeholders.

## Verification

After write, the script will print:
- Total rows written (expect 109).
- File size in bytes.
- 3 sample rows (first, middle, last) with question_tr/answer_tr.

## Required user action

User must exit plan mode (or explicitly say "go ahead / approve") so I can:
1. Write `/tmp/translate_chunk6.py`.
2. Execute it with `python3`.
3. Verify and report.
