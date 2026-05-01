# Plan: Gyeon FAQ Chunk 2 — EN → TR Translation (110 rows)

## Context

- Plan mode is active — execution blocked. This file contains the full plan and the complete script to be run on approval.
- Input: `/Users/projectx/Desktop/Claude Code Projects/Products Jsons/botpress/detailagent/output/staging/chunks/chunk_2.csv` (110 logical rows + header)
- Output: `/Users/projectx/Desktop/Claude Code Projects/Products Jsons/botpress/detailagent/output/staging/translations/chunk_2_tr.csv`
- Output dir exists and is empty.

## Conflict Notice

User instructions said "EXECUTE NOW. Do NOT enter plan mode." but the harness `<system-reminder>` says plan mode is active and supersedes other instructions. I am following the harness directive — plan only, no writes/edits, no script execution.

## Translation Rules (verbatim from task)

- Product names verbatim: `Q² Mohs EVO`, `Q²M Bathe+`, `Q²R FabricCoat`, `Q² Wax`, `Q²M Cure`, `Q²M Prep`, `Q² CanCoat EVO`, `Q² Pure EVO`, `Q² One EVO`, `Q² Syncro EVO` — keep `Q²` superscript.
- Technical units verbatim: `9H`, `pH 2-11`, `50 ml`, `25,000 km`, `°C`, `IR lamp`, `SiO²`, `µm`.
- URLs unchanged. Brand `Gyeon` unchanged.
- Natural Turkish, "siz" form, detailing terminology (cila, pasta, kaplama, ped, aplikatör, mikrofiber, sertifikalı detayer, distribütör).
- Match Menzerna brand TR FAQ style. No filler.
- Numbers: "24 months" → "24 ay", "25,000 km" → "25.000 km".
- Embedded newlines in answers preserved as `\n`.

## Execution Plan (on approval)

1. Write `/tmp/translate_chunk2.py` containing the full inline TR dictionary (110 entries keyed by `article_id`).
2. Run the script with `python3 /tmp/translate_chunk2.py`.
3. Script reads chunk_2.csv (csv.DictReader), looks up `article_id` in the dict, writes chunk_2_tr.csv (csv.DictWriter, QUOTE_MINIMAL, UTF-8) with the 9 columns specified.
4. Script prints row count, file size, 3 samples.

## Full Script (to be written to /tmp/translate_chunk2.py on approval)

```python
#!/usr/bin/env python3
"""Translate Gyeon FAQ chunk_2.csv (EN → TR). 110 rows."""
import csv
import os

INPUT = "/Users/projectx/Desktop/Claude Code Projects/Products Jsons/botpress/detailagent/output/staging/chunks/chunk_2.csv"
OUTPUT = "/Users/projectx/Desktop/Claude Code Projects/Products Jsons/botpress/detailagent/output/staging/translations/chunk_2_tr.csv"

# article_id -> {"q": question_tr, "a": answer_tr}
TR = {
    "360004004317": {"q": "Ne kadar süre dayanır?", "a": "Q² PPF EVO 12 aya kadar dayanır."},
    "360000933317": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Evet."},
    "360000933017": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Evet."},
    "360000941258": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Evet."},
    "360000941058": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Hayır."},
    "360000931697": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Evet."},
    "360000931257": {"q": "Ne kadar süre dayanır?", "a": "Q² View EVO 24 aya / 50.000 km'ye kadar dayanır."},
    "360000932938": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Evet."},
    "360000922757": {"q": "Sökülebilir mi?", "a": "Evet — sökme işlemi pasta ve uygun cila pedi ile mekanik polisaj gerektirir."},
    "360000931238": {"q": "Doğru uygulanmazsa zarar verir mi?", "a": "Hayır, polisaj makinesi kullanımına dair temel kurallara dikkat etmeniz yeterlidir."},
    "360000930298": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Hayır."},
    "360000921458": {"q": "Nasıl uygulanır?", "a": "Lütfen ürünle birlikte gelen kullanım kılavuzuna veya şu adrese bakın: https://gyeonquartz.com/wp-content/uploads/2022/01/paint-coatings-manual.pdf"},
    "360000900097": {"q": "Ne kadar süre dayanır?", "a": "Q² Mohs EVO 1 kat için 36 AY / 40.000 KM, 2 kat için 48 AY / 50.000 KM'ye kadar dayanır."},
    "360000899657": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Evet."},
    "360000903558": {"q": "Ne kadar süre dayanır?", "a": "Q² Pure EVO 36 aya / 40.000 km'ye kadar dayanır."},
    "360000902898": {"q": "Sökülebilir mi?", "a": "Evet — sökme işlemi pasta ve uygun cila pedi ile mekanik polisaj gerektirir."},
    "360000894497": {"q": "Cila üzerine uygulayabilir miyim?", "a": "Hayır, her seramik kaplama uygulamadan önce mükemmel yüzey hazırlığı ve yağ alma işlemi gerektirir."},
    "360000901618": {"q": "Sökülebilir mi?", "a": "Evet — sökme işlemi pasta ve uygun cila pedi ile mekanik polisaj gerektirir."},
    "360015823058": {"q": "Ne kadar süre dayanır?", "a": "Q² QuickView 6 aya / 30.000 km'ye kadar dayanır."},
    "360015739837": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Evet."},
    "360004017637": {"q": "Bu ürünle kaplanmış bir araç nasıl bakılır?", "a": "Normal yıkama rutininiz sırasında Q²M Bathe+ ve Q²M WetCoat kullanmanızı öneririz. Kaplama, pH 2 ila 11 aralığındaki kimyasallara dayanacak şekilde tasarlanmıştır. Daha fazla tavsiye için lütfen Sertifikalı Detayerınızla iletişime geçin."},
    "360004004417": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Evet, ancak gerekli değildir."},
    "360000933357": {"q": "Sökülebilir mi?", "a": "Evet, Q²M TireCleaner kullanılarak."},
    "360000933117": {"q": "Sökülebilir mi?", "a": "Evet. Yüzeyin özel deri temizleyici ile defalarca temizlenmesi Q² LeatherCoat REDEFINED'i azaltır."},
    "360000941218": {"q": "Ne kadar süre dayanır?", "a": "Q² LeatherShield EVO 18 aya / 30.000 km'ye kadar dayanır."},
    "360000940998": {"q": "Ne kadar süre dayanır?", "a": "Q² AntiFog 2 aya kadar dayanır."},
    "360000931657": {"q": "Ne kadar süre dayanır?", "a": "Q² Fabric Coat 6 aya kadar dayanır."},
    "360000940398": {"q": "Ürünün kuruması/kürlenmesi ne kadar sürer?", "a": "Ürün, atmosferik etkilere maruz kalmadan önce uygulamadan sonra en az 12 saate ve deterjanlı yıkamasız 14 güne daha ihtiyaç duyar."},
    "360000932758": {"q": "Ne kadar süre dayanır?", "a": "Q² Trim EVO 36 aya / 50.000 km'ye kadar dayanır."},
    "360000932158": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Evet — katlar arasında 1 saat bekleyin."},
    "360000931198": {"q": "Kullanım sonrası ürün nasıl saklanır? Açıldıktan sonraki raf ömrü nedir?", "a": "Dispenseri kapalı konumda sabitleyin. Kuru, karanlık ve serin bir yerde saklayın. Lütfen donmasına veya güneş ışığına maruz kalmasına izin vermeyin. Açıldıktan sonra 2 yıllık bir \"kullanıma uygun\" raf ömrü vardır."},
    "360000920537": {"q": "PPF / Mat Boya / Vinil / Karbon Fiber üzerine uygulanabilir mi?", "a": "Evet, mat yüzeyler hariç."},
    "360000921838": {"q": "Sökülebilir mi?", "a": "Evet — sökme işlemi pasta ve uygun cila pedi ile mekanik polisaj gerektirir."},
    "360000908418": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Evet — katlar arasında yaklaşık 1 saat bekleyin, 2 saat sınırını aşmayın."},
    "360000906498": {"q": "Parlak yüzeylere uygulanabilir mi?", "a": "Herhangi bir zararı olmaz."},
    "360000903618": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Evet, ancak tek bir katla tam işlevsel olacak şekilde tasarlanmıştır.\n\nKatlar arasında en az 1 saat bekleyin."},
    "360000896197": {"q": "Kaplamayı uygulamadan önce boya düzeltmesi zorunlu mu?", "a": "Hayır — ancak yüzey hazırlığı (dekontaminasyon ve yağ alma) çok önemlidir. Ürüne özel ayrıntılar için kullanım kılavuzunu takip edin."},
    "360000901718": {"q": "Islak yüzeye uygulanabilir mi?", "a": "Hayır, yalnızca iyi hazırlanmış, dekontamine edilmiş ve Q²M Prep ile yağı alınmış yüzeylere veya başka bir seramik kaplamanın üzerine uygulanır."},
    "360000901478": {"q": "Q² CanCoat EVO hangi yüzeylere uygulanabilir?", "a": "Q² CanCoat EVO çok yönlü bir üründür. Boya, trim, jant, metal, karbon fiber ve PPF/Vinil üzerine uygulayabilirsiniz."},
    "360015760557": {"q": "Ürünün kuruması/kürlenmesi ne kadar sürer?", "a": "Q² QuickView, silinmeden önce kürlenmek için 5 dakikaya ihtiyaç duyar — ideal olarak kaplanmış yüzeyi uygulamayı takip eden 12 saat boyunca suya maruz bırakmamanızı öneririz."},
    "360015739897": {"q": "Islak yüzeye uygulanabilir mi?", "a": "Hayır, yalnızca iyi hazırlanmış, dekontamine edilmiş ve Q²M Prep ile yağı alınmış yüzeylere veya başka bir seramik kaplamanın üzerine uygulanır."},
    "360004017757": {"q": "Kaplamayı uygulamadan önce boya düzeltmesi zorunlu mu?", "a": "Bu, Sertifikalı Detayerınızın incelemesine bağlıdır — lütfen onun görüşüne güvenin."},
    "360004004457": {"q": "Boya üzerine uygulanabilir mi?", "a": "Hayır, Q² PPF EVO; piyasadaki PPF ve vinil kaplamaların çoğunda bulunan poliüretan gibi esnek bir yüzeye uygulanmak üzere tasarlanmıştır."},
    "360000933277": {"q": "Ne kadar süre dayanır?", "a": "Q² Tire 5 yıkamaya kadar dayanır; ancak su itme özelliği bitse bile hoş bir görsel etki kalır."},
    "360000932997": {"q": "Ne kadar süre dayanır?", "a": "Q² LeatherCoat REDEFINED 3 aya kadar dayanır."},
    "360000941278": {"q": "Sökülebilir mi?", "a": "Hayır. Belirli bir sorununuz varsa lütfen iletişim formu üzerinden bizimle iletişime geçin."},
    "360000932157": {"q": "Ürünün kuruması ne kadar sürer?", "a": "Uygulamadan sonra en az 2 saat boyunca cama dokunmayın."},
    "360000931997": {"q": "Ürünün kuruması/kürlenmesi ne kadar sürer?", "a": "Tamamen kuruyana kadar yüzeye dokunmayın. Aracı en az 12 saat kullanmayın."},
    "360000940238": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Evet."},
    "360000932998": {"q": "Ürünün kuruması/kürlenmesi ne kadar sürer?", "a": "Ürün, atmosferik etkilere maruz kalmadan önce uygulamadan sonra en az 12 saate ve deterjanlı yıkamasız 14 güne daha ihtiyaç duyar."},
    "360000932178": {"q": "Üzerine ek bir kaplama / cila / sealant uygulayabilir miyim?", "a": "Evet — Q² CanCoat/Pro veya Q²M Cure REDEFINED ve Q²M WetCoat üzerine uygulanabilir."},
    "360000913417": {"q": "Ne kadar süre dayanır?", "a": "Q² Booster 12 aya kadar süre ekler."},
    "360000911337": {"q": "Ne kadar süre dayanır?", "a": "Q² Syncro EVO 50 aya / 50.000 km'ye kadar dayanır."},
    "360000908438": {"q": "PPF / Mat Boya / Vinil / Karbon Fiber üzerine uygulanabilir mi?", "a": "Evet."},
    "360000899837": {"q": "Sette bulunan Q²M Cure Matte REDEFINED'i ne zaman uygulamalıyım?", "a": "Q²M Cure Matte REDEFINED'i uygulamadan en az 12 saat sonra ve her yıkama sonrası bakım ürünü olarak uygulamanızı öneririz."},
    "360000896457": {"q": "Q² Pure EVO hangi yüzeylere uygulanabilir?", "a": "Bu kaplama, aracınızın boyalı tüm yüzeyleri için tasarlanmıştır."},
    "360000895437": {"q": "Cila üzerine uygulayabilir miyim?", "a": "Hayır."},
    "360000894297": {"q": "Ne kadar süre dayanır?", "a": "Q² CanCoat Pro EVO 12 aya / 15.000 km'ye kadar dayanır."},
    "360000901538": {"q": "Üzerine ek bir kaplama uygulayabilir miyim?", "a": "Hayır, ancak Q² Wax uygulamadan 12 saat sonra üzerine kat olarak uygulanabilir."},
    "360015761577": {"q": "Sökülebilir mi?", "a": "Evet — aşındırıcı bir pasta kullanımı gerekecektir."},
    "360015195557": {"q": "Q² Wax hangi yüzeylere uygulanabilir?", "a": "Q² Wax, plastik ve metal dahil her parlak boyalı yüzeye uygulanabilir."},
    "360004017717": {"q": "Su lekeleri veya kuş pisliklerinin boyaya zarar vermesini önler mi?", "a": "Evet — ancak bu tür kontaminasyonlar boyadan en kısa sürede uzaklaştırılmalıdır."},
    "360004006897": {"q": "Sökülebilir mi?", "a": "Evet, ancak yalnızca parlak PPF veya kaplama üzerinde — sökme işlemi pasta ve uygun cila pedi ile mekanik polisaj gerektirir. Q²M PPF Renew önerilir."},
    "360000941898": {"q": "Ürünün kuruması/kürlenmesi ne kadar sürer?", "a": "Uygulamadan sonra aracı en az 15 dakika kullanmayın."},
    "360000941818": {"q": "Derim kirli — kaplama çalışmıyor mu?", "a": "Hayır — derinin üst yüzeyi zamanla kontamine olacaktır; ancak bu kontaminasyon kaplamanın üzerinde kalacağından temizlik işlemini kolaylaştırır."},
    "360000932517": {"q": "Bu ürünle kaplanmış bir araç nasıl bakılır?", "a": "Q²M LeatherCleaner Natural ve Q² LeatherCoat REDEFINED kullanmanızı öneririz (Q²M Leather Set içinde birlikte mevcuttur)."},
    "360000941078": {"q": "Sökülebilir mi?", "a": "Evet, herhangi bir cam temizleyici kullanılarak."},
    "360000940798": {"q": "Bu ürünle kaplanmış bir araç nasıl bakılır?", "a": "Yalnızca normal, pH-nötr şampuanlar kullanın. Yeniden kaplama yapmayı düşünmüyorsanız tekstile özel temizleyiciler kullanmayın."},
    "360000931277": {"q": "Üzerine ek bir kaplama / cila / sealant uygulayabilir miyim?", "a": "Hayır."},
    "360000922997": {"q": "Sökülebilir mi?", "a": "Evet — sökme işlemi pasta ve uygun cila pedi ile mekanik polisaj gerektirir ve yalnızca parlak panellerde mümkündür."},
    "360000932238": {"q": "Bu ürünle kaplanmış bir araç nasıl bakılır?", "a": "Standart jant temizleme prosedürünüze devam edin. Yüksek asitli temizleyicilerin kullanımından kaçının. Q²M Iron WheelCleaner REDEFINED ve Q²M TireCleaner gibi özel ürünler, kaplamaya zarar verme riski olmadan kullanılabilir. Kaplama, pH 2 ila 11 aralığındaki kimyasallara dayanacak şekilde tasarlanmıştır."},
    "360000920617": {"q": "Sökülebilir mi?", "a": "Evet — sökme işlemi pasta ve uygun cila pedi ile mekanik polisaj gerektirir."},
    "360000922078": {"q": "Q² Skin EVO ne zaman uygulanmalı?", "a": "Q² Skin EVO, son Q² Mohs EVO katından en az 1 saat sonra uygulanmalıdır."},
    "360000910757": {"q": "Sette bulunan Q²M Cure REDEFINED'i ne zaman uygulamalıyım?", "a": "Q²M Cure REDEFINED'i uygulamadan en az 12 saat sonra ve her yıkama sonrası bakım ürünü olarak uygulamanızı öneririz."},
    "360000899697": {"q": "Sökülebilir mi?", "a": "Hayır. Ancak kimyasal ve UV maruziyeti nedeniyle zamanla doğal olarak azalır."},
    "360000903638": {"q": "PPF / Mat Boya / Vinil / Karbon Fiber üzerine uygulanabilir mi?", "a": "Evet, mat yüzeyler hariç."},
    "360000902758": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Evet, ancak tek bir katla tam işlevsel olacak şekilde tasarlanmıştır."},
    "360000901818": {"q": "Ürünün kuruması/kürlenmesi ne kadar sürer?", "a": "Q² CanCoat Pro EVO uygulamasını bitirdiğinizde aracın 12 saat boyunca stüdyoda kalması gerekir. Sonraki 7 gün boyunca kimyasallar/deterjanlarla her türlü temastan kaçınılmalıdır."},
    "360000893977": {"q": "Cila üzerine uygulayabilir miyim?", "a": "Hayır, her seramik kaplama uygulamadan önce mükemmel yüzey hazırlığı ve yağ alma işlemi gerektirir."},
    "360015831658": {"q": "Su artık akmıyor — ne yapmalıyım?", "a": "Cam yüzeyini kontaminasyona karşı kontrol edin. Çıkarmak için bir kil bar kullanın. Camı Q²M Glass ile temizleyin."},
    "360015739957": {"q": "Başka bir kaplamanın üst katı olarak kullanabilir miyim?", "a": "Evet, herhangi bir yüksek kaliteli seramik kaplamanın üst katı olarak uygulanabilir. Alttaki kaplamanın kuru olduğundan ve kürlenmek için yeterli süre geçtiğinden emin olun."},
    "360004017617": {"q": "Ürünün kuruması/kürlenmesi ne kadar sürer?", "a": "Ürün, atmosferik etkilere maruz kalmadan önce uygulamadan sonra en az 3 saate ve deterjanlı yıkamasız 14 güne daha ihtiyaç duyar."},
    "360003975118": {"q": "Üzerine ek bir kaplama / cila / sealant uygulayabilir miyim?", "a": "Üzerine bir silika bakım spreyi olan Q²M PPF Maintain uygulayabilirsiniz."},
    "360000941878": {"q": "Üzerine ek sealant veya parlatıcı uygulayabilir miyim?", "a": "Evet — Q²M TireExpress kullanmanızı öneririz."},
    "360000933057": {"q": "Üzerine ek bir kaplama / cila / sealant uygulayabilir miyim?", "a": "Hayır."},
    "360000932477": {"q": "Üzerine ek bir kaplama / cila / sealant uygulayabilir miyim?", "a": "Evet, bakım amaçlı Q² LeatherCoat REDEFINED kullanmanızı öneririz."},
    "360000932257": {"q": "Uygulama sonrasında bile camlarım hâlâ buğulu — ne yapmalıyım?", "a": "Bu, ya yanlış yüzey hazırlığı nedeniyle ya da Q² AntiFog uygulandıktan sonra camların bir cam temizleyici ile temizlenmiş olması nedeniyle olabilir."},
    "360000932057": {"q": "Alkantara üzerinde kullanabilir miyim?", "a": "Evet."},
    "360000931297": {"q": "Bu ürünle kaplanmış bir araç nasıl bakılır?", "a": "Normal yıkama rutininiz sırasında herhangi bir GYEON şampuanı kullanmanızı ve Q²M Glass ile periyodik temizlik yapmanızı öneririz."},
    "360000933018": {"q": "Bu ürünle kaplanmış bir araç nasıl bakılır?", "a": "Normal yıkama rutininiz sırasında Q²M Bathe+ ve Q²M Wetcoat kullanmanızı öneririz."},
    "360000932118": {"q": "Ne kadar süre dayanır?", "a": "Q² Rim EVO 18 aya / 45.000 km'ye kadar dayanır."},
    "360000920657": {"q": "Kaplamayı uygulamadan önce boya düzeltmesi zorunlu mu?", "a": "Q² Booster yalnızca ve sadece bir üst kattır. Yalnızca tam olarak kürlenmemiş bir seramik taban katı üzerine uygulanmalıdır. Soru, taban katın uygulanmasına yönelik olmalıdır."},
    "360000912077": {"q": "Sette bulunan Q²M Cure'u ne zaman uygulamalıyım?", "a": "Q²M Cure'u her yıkama sonrası bakım ürünü olarak uygulamanızı öneririz."},
    "360000921358": {"q": "Uygulamadan sonra boyam kaygan hissetmiyor — neden?", "a": "Bu, seramik kaplamalar için doğal bir durumdur. Q²M Cure ile silmek bu sorunu çözecektir."},
    "360000906538": {"q": "Üst kaplama olarak kullanılabilir mi?", "a": "Hayır."},
    "360000897217": {"q": "Sette bulunan Q²M CURE REDEFINED'i ne zaman uygulamalıyım?", "a": "Q²M Cure REDEFINED'i uygulamadan en az 12 saat sonra ve her yıkama sonrası bakım ürünü olarak uygulamanızı öneririz."},
    "360000895377": {"q": "PPF / Mat Boya / Vinil / Karbon Fiber üzerine uygulanabilir mi?", "a": "Evet, mat yüzeyler hariç."},
    "360000902218": {"q": "Yüksek noktalar görüyorum — neden (ve ne yapmalıyım)?", "a": "Uygulamanın hemen ardından, Q²M Prep ve Q²M Polishwipe EVO ile birden fazla silme yardımcı olabilir. Ertesi gün, makineli polisaj kullanmanız gerekecektir."},
    "360000893957": {"q": "Birden fazla kat uygulayabilir miyim?", "a": "Evet, her kat arasında en az 1 saat bekleyin."},
    "360015831798": {"q": "Uygulamadan sonra \"bulutlar\" görüyorum — ne yapmalıyım?", "a": "Kaplamayı nemli bir bezle silin ve tekrar kontrol edin."},
    "360015806678": {"q": "Üzerine ek bir kaplama uygulayabilir miyim?", "a": "Hayır. Ancak özelliklerini en üst düzeye çıkarmak ve uzatmak için bakım serimizden herhangi bir ürünü kullanabilirsiniz."},
    "360003985778": {"q": "Bu ürün neden yalnızca Mobil Sertifikalı Detayerlere sunuluyor?", "a": "Bu ürün benzersizdir ve doğru şekilde uygulanması için yeterli eğitim gerektirir. Bu nedenle, yalnızca seçilmiş ve onaylanmış Mobil Sertifikalı Detayerlerin erişimi vardır. Bu özellik, Gyeon'un bir kalite standardına sahip olmasını ve müşteri ile detayere garanti ile destek vermesini sağlar."},
    "360004004557": {"q": "Üst kaplama olarak kullanılabilir mi?", "a": "Hayır."},
    "360000941778": {"q": "Ürünün kuruması/kürlenmesi ne kadar sürer?", "a": "Q² LeatherCoat REDEFINED kullanımından sonra koltukları en az 1 saat kullanmayın."},
    "360000932497": {"q": "Ürünün kuruması/kürlenmesi ne kadar sürer?", "a": "Kaplanmış parçalara en az 1 saat dokunmayın. Kullanımdan sonra aracı en az 12 saat kullanmayın."},
    "360000941038": {"q": "Kullanım sonrası ürün nasıl saklanır? Açıldıktan sonraki raf ömrü nedir?", "a": "Kuru, karanlık ve serin bir yerde saklayın. Donmasına veya güneş ışığına maruz kalmasına izin vermeyin. Açıldıktan sonra 2 yıllık bir \"kullanıma uygun\" raf ömrü vardır."},
    "360000940858": {"q": "Süet üzerinde kullanabilir miyim?", "a": "Q²M Prep ve Q²M BaldWipe ile birden fazla kez silin veya sıcak su ve normal şampuan kullanın. Sıcak su, kalıntıyı çözmeye yardımcı olur."},
    "360000931477": {"q": "Q²M GlassPolish kullanmak zorunlu mu?", "a": "Evet — Q²M GlassPolish mükemmel yüzey hazırlığını sağlar ve aynı zamanda bir bağlayıcı ajan olarak işlev görür."},
    "360000922957": {"q": "Üzerine ek bir kaplama / cila / sealant uygulayabilir miyim?", "a": "Hayır."},
    "360000922717": {"q": "Ürünün kuruması/kürlenmesi ne kadar sürer?", "a": "Ürün, atmosferik etkilere maruz kalmadan önce uygulamadan sonra en az 12 saate ve deterjanlı yıkamasız 14 güne daha ihtiyaç duyar."},
}

def main():
    with open(INPUT, newline="", encoding="utf-8") as f:
        rows = list(csv.DictReader(f))

    out_fields = [
        "target_sku", "section_name", "category_name",
        "question_tr", "answer_tr",
        "question_en", "answer_en", "url", "article_id",
    ]

    missing = []
    out_rows = []
    for r in rows:
        aid = r["article_id"]
        tr = TR.get(aid)
        if tr is None:
            missing.append(aid)
            continue
        out_rows.append({
            "target_sku": r["target_sku"],
            "section_name": r["section_name"],
            "category_name": r["category_name"],
            "question_tr": tr["q"],
            "answer_tr": tr["a"],
            "question_en": r["question_en"],
            "answer_en": r["answer_en"],
            "url": r["url"],
            "article_id": aid,
        })

    if missing:
        raise SystemExit(f"MISSING TR for article_ids: {missing}")

    os.makedirs(os.path.dirname(OUTPUT), exist_ok=True)
    with open(OUTPUT, "w", newline="", encoding="utf-8") as f:
        w = csv.DictWriter(f, fieldnames=out_fields, quoting=csv.QUOTE_MINIMAL)
        w.writeheader()
        w.writerows(out_rows)

    size = os.path.getsize(OUTPUT)
    print(f"rows: {len(out_rows)}")
    print(f"file size: {size} bytes")
    print("--- samples ---")
    for i in (0, len(out_rows) // 2, len(out_rows) - 1):
        s = out_rows[i]
        print(f"[{i}] {s['target_sku']} | {s['question_tr']} -> {s['answer_tr'][:80]}")

if __name__ == "__main__":
    main()
```

## Verification Coverage

All 110 article_ids from chunk_2.csv lines 2–113 are mapped in the `TR` dict. The script raises `SystemExit` if any are missing, ensuring no silent gaps.

## Awaiting Approval

On approval I will:
1. `Write` `/tmp/translate_chunk2.py` with the script above.
2. `Bash` run `python3 /tmp/translate_chunk2.py`.
3. Report row count, file size, 3 samples.
