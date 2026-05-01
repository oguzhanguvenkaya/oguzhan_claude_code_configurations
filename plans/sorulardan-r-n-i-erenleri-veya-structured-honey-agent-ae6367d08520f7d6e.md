# Plan: Translate Gyeon FAQ chunk_4 (110 rows) EN→TR

## Goal
Produce `output/staging/translations/chunk_4_tr.csv` with 110 Turkish-translated FAQ Q&A pairs from `output/staging/chunks/chunk_4.csv`.

## Conflict Notice
The user instruction says "EXECUTE NOW. Do NOT enter plan mode." However, the system-reminder enforces plan mode and explicitly states it supersedes any other instructions. Therefore I am presenting the plan; the user must exit plan mode (e.g. accept the plan) for me to actually write the script and CSV.

## Inputs verified
- `output/staging/chunks/chunk_4.csv` exists (110 logical rows; some answers contain embedded newlines inside quoted CSV fields).
- Output directory `output/staging/translations/` does NOT yet exist; will be created.

## Execution Steps (after plan accepted)

1. **Create script** at `/tmp/translate_chunk4.py`:
   - Read input CSV with `csv.DictReader` (handles quoted multi-line fields).
   - Build a dict `TRANSLATIONS: dict[str, dict]` keyed by `article_id` (string), values `{"question_tr": str, "answer_tr": str}`. All 110 entries hand-translated inline.
   - Iterate rows in original order. For each row, look up by `article_id`. If missing → raise (no placeholders allowed).
   - Write output CSV with `csv.DictWriter`, `quoting=csv.QUOTE_MINIMAL`, encoding `utf-8`, columns:
     `target_sku, section_name, category_name, question_tr, answer_tr, question_en, answer_en, url, article_id`.
   - Preserve embedded `\n` literally (Python `csv` module handles automatically).
   - `mkdir -p output/staging/translations` before writing.

2. **Translation rules applied**:
   - Verbatim: product names with `Q²` superscript (e.g. `Q²M Bathe+`, `Q² Mohs EVO`, `Q²R FabricCoat`, `Q²M Cure REDEFINED`), `Gyeon`, URLs.
   - Verbatim units: `9H`, `pH 2-11`, `50 ml`, `25,000 km` → `25.000 km` (TR locale), `°C`, `IR lamp`, `SiO²`/`SiO₂`, `µm`, dilution ratios `1:5`, `1:15`, `1:256`.
   - Time: `24 months` → `24 ay`.
   - Tone: formal "siz" form, professional detailing terminology (kaplama, parlatma, dekontaminasyon, hidrofobik, mikrofiber, vb.).
   - No filler additions.

3. **Run** the script: `python3 /tmp/translate_chunk4.py`.

4. **Verify**:
   - Row count = 110.
   - File size > 0.
   - Print 3 sample translated rows.

## Risks
- 110 manual translations is a large hand-written payload (~30-60 KB of Turkish text). Need to enumerate every `article_id` from the CSV (already inspected via Read; will re-read all 110 if needed before writing the script).
- Multi-line answers (e.g. ECOWash, Scents, GlassPolish) must keep `\n` separators.

## Deliverables
- `/tmp/translate_chunk4.py`
- `output/staging/translations/chunk_4_tr.csv` (110 rows)
- Final report: row count, file size, 3 samples (under 200 words).
