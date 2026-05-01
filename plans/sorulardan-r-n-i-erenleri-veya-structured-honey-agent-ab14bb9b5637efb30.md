# Plan: Translate Gyeon FAQ chunk_5 (110 rows) EN → TR

## Status

Plan mode is currently active. The user requested immediate execution ("EXECUTE NOW"), but per the system reminder I cannot run write tools. I have the full source data analyzed and the complete translation prepared in this plan. On user approval (exit plan mode), I will:

1. Write `/tmp/translate_chunk5.py` with the inline `article_id → {question_tr, answer_tr}` mapping below.
2. Run it: `python3 /tmp/translate_chunk5.py`.
3. Report rows count, file size, and 3 sample rows.

## Input

- Path: `output/staging/chunks/chunk_5.csv` (117 lines incl. header + multi-line embedded rows = 110 logical rows).
- All rows belong to `Q²M Collection`.
- Embedded newlines exist in article_id `18565739075100` (Q²M ECOWash agitate).

## Output

- Path: `output/staging/translations/chunk_5_tr.csv`
- Columns: `target_sku, section_name, category_name, question_tr, answer_tr, question_en, answer_en, url, article_id`
- `csv.DictWriter`, `QUOTE_MINIMAL`, UTF-8.

## Translation Rules Applied

- Product names verbatim with `Q²` superscript (e.g. `Q²M Bathe+`, `Q²M Smoothie`, `Q² Rim`, `Q² AntiFog`).
- Units verbatim: `15ml`, `20ml`, `1:5`, `1:15`, `1:256`, `1:150`, `1:20`, `1:10`, `pH 4-9`.
- "24 months" → "24 ay"; "25,000 km" → "25.000 km"; "3 years" → "3 yıl"; "6 weeks" → "6 hafta"; "30 seconds" → "30 saniye"; "2 washes" → "2 yıkama"; "over a week" → "bir haftadan fazla".
- URLs unchanged. `Gyeon` unchanged.
- "siz" form, detailing terms (cila, pasta, kaplama, ped, aplikatör, mikrofiber, durulama, dekontaminasyon).
- Embedded newlines kept (article_id `18565739075100`).

## Translation Mapping (110 entries)

```python
TRANSLATIONS = {
    "360000972658": {
        "question_tr": "Nasıl kullanılır?",
        "answer_tr": "Q²M Bathe+ kullanmanın 3 farklı yolu vardır: - 1. Bir kova suya 15ml Q²M Bathe+ dökün - Çözeltiyi karıştırın - Çözeltiyi aracınıza aktarmak için Q²M Smoothie kullanın - yıkama rutininize devam edin - 2. Q²M Bathe+'yı doğrudan Q²M Smoothie üzerine dökün ve yıkama işleminize panel panel devam edin - her panel için tekrarlayın - 3. Foam Gun'unuza 20ml Q²M Bathe+ dökün - Aracın tamamına püskürtün ve durulayın.",
    },
    "360000972398": {
        "question_tr": "Nasıl kullanılır?",
        "answer_tr": "Bir kova suya 15-20ml Q²M Bathe dökün - Çözeltiyi karıştırın - Çözeltiyi aracınıza aktarmak için Q²M Smoothie kullanın - yıkama rutininize devam edin.",
    },
    "360000961217": {
        "question_tr": "Nasıl kullanılır?",
        "answer_tr": "Küçük bir bölüme püskürtün - Ürünün kurumasına izin vermeyin - Durulayın.",
    },
    "360000972278": {
        "question_tr": "Nasıl kullanılır?",
        "answer_tr": "Aracın tüm yüzeylerine bolca püskürtün, jantlar için ürünü ovalayarak işleyin - reaksiyona girmesini bekleyin ve durulayın. Kurumasına izin vermeyin - Sıcak yüzeylere uygulamayın.",
    },
    "360000972178": {
        "question_tr": "Nasıl kullanılır?",
        "answer_tr": "Kullanmadan önce seyreltmenizi öneririz - Çalışacağınız panele bolca püskürtün - kil işlemine devam edin - fazla ürünü mikrofiber ile alın.",
    },
    "360000972058": {
        "question_tr": "Nasıl kullanılır?",
        "answer_tr": "Elinizin 1/3'ünü kaplayacak büyüklükte küçük bir kil parçası şekillendirin - Dekontamine etmek istediğiniz panele bolca Q²M ClayLube püskürtün - Kil parçasını seçili alan üzerinde düz hareketlerle gezdirin - Fazla yağlayıcıyı bir mikrofiber ile silin - sonucu değerlendirin - dokunduğunuzda hâlâ kontaminasyon hissediyorsanız tekrarlayın - Kil aşırı kontaminasyonla dolduğunda, çalıştığınız yüzeye zarar vermemek için başka bir parça kullanın.",
    },
    "360000959857": {
        "question_tr": "Nasıl kullanılır?",
        "answer_tr": "Küçük bir bölüme püskürtün - Yumuşak bir mikrofiber kullanın ve etkilenen alanı birkaç kez silin. Gerekirse işlemi tekrarlayın - Özellikle camda olmak üzere ürünün kurumasına izin vermeyin.",
    },
    "360000959757": {
        "question_tr": "Nasıl kullanılır?",
        "answer_tr": "Aracınızın soğuk ve kirlenmiş yüzeyine bolca püskürtün - Doğrudan güneş ışığında püskürtmeyin - Kurumasına izin vermeyin - Durulayın - Gerekirse tekrar uygulayın.",
    },
    "360000971578": {
        "question_tr": "Nasıl kullanılır?",
        "answer_tr": "Foam Cannon veya Foamer Sprayer'ınıza 1:5 ile 1:15 arasında seyreltin. Aracınızın dış yüzeyinin tamamına püskürtün - Kiri çözmesi için 3-5 dakika bekleyin - Durulayın ve yıkama rutininize devam edin.",
    },
    "360000959397": {
        "question_tr": "Nasıl uygulanır?",
        "answer_tr": "Bir mikrofibere bolca püskürtün - Q²M SoftWipe yardımıyla ürünü boyalı yüzeye uygulayın - Sıcak yüzeylere püskürtmekten kaçının.",
    },
    "360000945898": {
        "question_tr": "Nasıl uygulanır?",
        "answer_tr": "Lütfen şişe üzerindeki kullanım talimatına başvurun.",
    },
    "360000945578": {
        "question_tr": "Nasıl uygulanır?",
        "answer_tr": "Derinizin küçük bir bölümüne bolca püskürtün - Ürünü Q²M LeatherBrush yardımıyla deri yüzeyine fırçalayın - tüm yüzey kaplandığında fazla ürünü mikrofiber ile alın - sonucu değerlendirin - gerekirse tekrar uygulayın.",
    },
    "360000943418": {
        "question_tr": "Nasıl uygulanır?",
        "answer_tr": "Derinizin küçük bir bölümüne bolca püskürtün - Ürünü Q²M LeatherBrush yardımıyla deri yüzeyine fırçalayın - tüm yüzey kaplandığında fazla ürünü mikrofiber ile alın - sonucu değerlendirin - gerekirse tekrar uygulayın.",
    },
    "360000934057": {
        "question_tr": "Nasıl uygulanır?",
        "answer_tr": "Tercih ettiğiniz aplikatöre bir pompa ürün boşaltın (Q²M MF Applicator kullanabilirsiniz). Ürünü lastiğinizin yan yüzeyine ovarak uygulayın. Hem ıslak hem kuru lastiklerde kullanılabilir. En iyi yüzey hazırlığı için Q²M TireCleaner kullanmanızı öneririz.",
    },
    "360000933857": {
        "question_tr": "Nasıl uygulanır?",
        "answer_tr": "Lastiğinizin lastik yan yüzeyine bolca püskürtün, kısa bir süre reaksiyona girmesini bekleyin ve ovalayın - reaksiyona girmesini bekleyin ve durulayın. Kurumasına izin vermeyin - Sıcak yüzeylere uygulamayın - Kirli lastiklerde kuru uygulama öneririz.",
    },
    "360000942198": {
        "question_tr": "Nasıl uygulanır?",
        "answer_tr": "Q²M PPF Renew'ü eksantrik polisaj makinesi ve Q²M Eccentric Finish Pad ile uygulamanızı öneririz. Az miktarda ürün kullanın ve düşük hızda çalışın.",
    },
    "18582186095644": {
        "question_tr": "Rotary mı DA mı?",
        "answer_tr": "Her ikisi de kullanılabilir.",
    },
    "18581367260572": {
        "question_tr": "Q²M DeFrost'u kış cam suyu olarak kullanabilir miyim?",
        "answer_tr": "Hayır.",
    },
    "18579518608412": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Bu çözelti, mikrofiber yardımıyla silme hareketi gerektirir.",
    },
    "18565735656604": {
        "question_tr": "Q²M ECOWash'u şampuanımla karıştırabilir miyim?",
        "answer_tr": "Hayır, Q²M ECOWash tek başına ürün olarak kullanılmalıdır.",
    },
    "18564492935580": {
        "question_tr": "Q²M TotalRemover boyama işlemime zarar verir/değiştirir mi?",
        "answer_tr": "Hayır, tüm boyalı yüzeyler için güvenlidir.",
    },
    "18563965019036": {
        "question_tr": "Birden fazla kat uygulayabilir miyim?",
        "answer_tr": "Evet.",
    },
    "5743883311889": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Bu çözelti, mikrofiber veya fırça yardımıyla silme hareketi gerektirir.",
    },
    "5743111956625": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Evet, işleyebilirsiniz.",
    },
    "5742177894673": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Bu çözelti mekanik işlem gerektirmez, ancak yapılabilir.",
    },
    "5741177647505": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Evet.",
    },
    "360004018577": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Evet.",
    },
    "360003986218": {
        "question_tr": "Ne kadar süre dayanır?",
        "answer_tr": "Q²M PPF Maintain REDEFINED 6 haftaya kadar dayanır.",
    },
    "360000983277": {
        "question_tr": "Ürün kullanıldıktan sonra nasıl saklanır? Açıldıktan sonra raf ömrü nedir?",
        "answer_tr": "Dispenseri kapalı pozisyonda sabitleyin. Kuru, karanlık ve serin bir yerde saklayın. Donmasına izin vermeyin ve güneş ışığına maruz bırakmayın. Açıldıktan sonra \"kullanıma uygun\" raf ömrü 3 yıldır.",
    },
    "360000983237": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Bu çözelti, mikrofiber veya Q²M MF aplikatör yardımıyla silme hareketi gerektirir.",
    },
    "360000982977": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Bu çözelti, mikrofiber yardımıyla silme hareketi gerektirir.",
    },
    "360000991918": {
        "question_tr": "Birden fazla kat uygulayabilir miyim?",
        "answer_tr": "Evet",
    },
    "360000991598": {
        "question_tr": "Ne kadar süre dayanır?",
        "answer_tr": "Q²M Cure Matte REDEFINED 6 haftaya kadar dayanır.",
    },
    "360000991378": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Hayır, sadece püskürt ve durula formülüdür.",
    },
    "360000991278": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Hayır, sadece püskürt ve durula formülüdür.",
    },
    "360000981597": {
        "question_tr": "Rotary mı DA mı?",
        "answer_tr": "Her iki tip polisaj makinesiyle de kullanılabilir.",
    },
    "360000990938": {
        "question_tr": "Rotary mı DA mı?",
        "answer_tr": "Her iki tip polisaj makinesiyle de kullanılabilir.",
    },
    "360000979977": {
        "question_tr": "Rotary mı DA mı?",
        "answer_tr": "Her iki tip polisaj makinesiyle de kullanılabilir.",
    },
    "360000962197": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Bu çözelti, mikrofiber yardımıyla silme hareketi gerektirir.",
    },
    "360000961937": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Evet",
    },
    "360000961477": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Evet",
    },
    "360000961317": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Bu çözelti, mikrofiber yardımıyla silme hareketi gerektirebilir.",
    },
    "360000961157": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Bu çözelti mekanik işlem gerektirmez, ancak yapılabilir.",
    },
    "360000972238": {
        "question_tr": "Ürün kullanıldıktan sonra nasıl saklanır? Açıldıktan sonra raf ömrü nedir?",
        "answer_tr": "Dispenseri kapalı pozisyonda sabitleyin. Kuru, karanlık ve serin bir yerde saklayın. Donmasına izin vermeyin ve güneş ışığına maruz bırakmayın. Açıldıktan sonra \"kullanıma uygun\" raf ömrü 3 yıldır.",
    },
    "360000960817": {
        "question_tr": "Q²M Claylube kullanmak zorunda mıyım?",
        "answer_tr": "Yağlayıcı kullanmanızı öneririz.",
    },
    "360000971798": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Bu çözelti, mikrofiber yardımıyla silme hareketi gerektirebilir.",
    },
    "360000971738": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Bu çözelti mekanik işlem gerektirmez, ancak derin oturmuş kir tekrar uygulama gerektirebilir.",
    },
    "360000959717": {
        "question_tr": "Q²M Foam'u durulamamdan sonra arabam hâlâ kirli - Neden?",
        "answer_tr": "Bu çözelti, gevşek kiri çözmek ve yıkama işlemini daha güvenli hâle getirmek için tasarlanmıştır.",
    },
    "360000971458": {
        "question_tr": "Birden fazla kat uygulayabilir miyim?",
        "answer_tr": "Evet",
    },
    "360000971298": {
        "question_tr": "Doğru uygulanmazsa zarar verir mi?",
        "answer_tr": "Mekanik bir işlem söz konusu olduğundan, uygulama öncesi polisaj makinesini doğru kullanmayı bilmeniz gerekir.",
    },
    "360000945798": {
        "question_tr": "Doğru uygulanmazsa iç döşememe zarar verir mi?",
        "answer_tr": "Hayır - lütfen kullanım kılavuzuna başvurun.",
    },
    "360000945438": {
        "question_tr": "Birden fazla kat uygulayabilir miyim?",
        "answer_tr": "Evet",
    },
    "360000934137": {
        "question_tr": "Birden fazla kat uygulayabilir miyim?",
        "answer_tr": "Evet",
    },
    "360000934017": {
        "question_tr": "Jantlarıma zarar verir mi?",
        "answer_tr": "Hayır. Çizgi veya leke oluşmasını önlemek için ürünü asla sıcak jantlara uygulamayın.",
    },
    "360000933717": {
        "question_tr": "Hangi pedi kullanmalıyım?",
        "answer_tr": "Q²M Eccentric Finish Pad kullanmanızı öneririz.",
    },
    "18581245585948": {
        "question_tr": "Q²M DeFrost'u doğrudan kar ve buz üzerine uygulayabilir miyim?",
        "answer_tr": "Evet, Q²M DeFrost herhangi bir hasara neden olmaz.",
    },
    "18579531750684": {
        "question_tr": "Q²M Glass+ çizgi bıraktı - Neden?",
        "answer_tr": "Çok fazla Q²M Glass+ kullanımı çizgilere yol açar.",
    },
    "18566524990876": {
        "question_tr": "Ne kadar süre dayanır?",
        "answer_tr": "Paspas veya birlikte verilen askı üzerine bir sıkım, bir haftadan fazla dayanır.",
    },
    "18565739075100": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Evet.\n\nSusuz yıkama için: 1:15/1:10 oranında seyreltin\n\nDurulamasız için: 1:256/1:150 oranında seyreltin\n\nKuruma yardımcısı için: 1:20 oranında seyreltin",
    },
    "18564812313628": {
        "question_tr": "Çözeltiyi mekanik olarak işlemem gerekir mi?",
        "answer_tr": "Evet. Ürünü Q²M SilkMitt EVO gibi özel bir eldivenle işleyin. Paneli durulamadan önce 30 saniye beklemesine izin verin.",
    },
    "18564000893724": {
        "question_tr": "Doğru uygulanmazsa zarar verir mi?",
        "answer_tr": "Hayır.",
    },
    "5743822457873": {
        "question_tr": "Ürün kullanıldıktan sonra nasıl saklanır? Açıldıktan sonra raf ömrü nedir?",
        "answer_tr": "Dispenseri kapalı pozisyonda sabitleyin. Kuru, karanlık ve serin bir yerde saklayın. Donmasına izin vermeyin ve güneş ışığına maruz bırakmayın. Açıldıktan sonra \"kullanıma uygun\" raf ömrü 3 yıldır.",
    },
    "5743088672145": {
        "question_tr": "Q²M APC boyama veya kaplamama zarar verir/değiştirir mi?",
        "answer_tr": "Çok yüksek konsantrasyonda kullanılır ve/veya kurumasına izin verilirse zarar verebilir.",
    },
    "5741669725841": {
        "question_tr": "Q²M Iron WheelCleaner REDEFINED'ı şampuanımla karıştırabilir miyim?",
        "answer_tr": "Hayır.",
    },
    "5741156525969": {
        "question_tr": "Q²M RestartWash boyama veya kaplamama zarar verir/değiştirir mi?",
        "answer_tr": "Hayır.",
    },
    "360003986738": {
        "question_tr": "Q²M PPF Wash boyama veya kaplamama zarar verir/değiştirir mi?",
        "answer_tr": "Hayır",
    },
    "360003986278": {
        "question_tr": "Birden fazla kat uygulayabilir miyim?",
        "answer_tr": "Evet.",
    },
    "360000992458": {
        "question_tr": "Ürün kullanıldıktan sonra nasıl saklanır? Açıldıktan sonra raf ömrü nedir?",
        "answer_tr": "Dispenseri kapalı pozisyonda sabitleyin. Kuru, karanlık ve serin bir yerde saklayın. Donmasına izin vermeyin ve güneş ışığına maruz bırakmayın. Açıldıktan sonra \"kullanıma uygun\" raf ömrü 3 yıldır.",
    },
    "360000992198": {
        "question_tr": "Q²M Glass çizgi bıraktı - Neden?",
        "answer_tr": "Çok fazla ürün kullanımı çizgilere yol açar.",
    },
    "360000991938": {
        "question_tr": "Üzerine ek bir kaplama uygulayabilir miyim?",
        "answer_tr": "Hayır",
    },
    "360000982277": {
        "question_tr": "Birden fazla kat uygulayabilir miyim?",
        "answer_tr": "Evet",
    },
    "360000991358": {
        "question_tr": "Q²M WetCoat Essence boyama veya kaplamama zarar verir/değiştirir mi?",
        "answer_tr": "Hayır, ancak kurumasına izin verilir veya sıcak yüzeye uygulanırsa çizgi/yüksek nokta bırakabilir.",
    },
    "360000981717": {
        "question_tr": "Q²M WetCoat boyama veya kaplamama zarar verir/değiştirir mi?",
        "answer_tr": "Hayır, ancak kurumasına izin verilir veya sıcak yüzeye uygulanırsa çizgi/yüksek nokta bırakabilir.",
    },
    "360000981577": {
        "question_tr": "Hangi ped kombinasyonu kullanılmalı?",
        "answer_tr": "Q²M Polish kullanmanızı öneririz.",
    },
    "360000990898": {
        "question_tr": "Hangi ped kombinasyonu kullanılmalı?",
        "answer_tr": "Q²M Cut kullanmanızı öneririz.",
    },
    "360000979957": {
        "question_tr": "Hangi ped kombinasyonu kullanılmalı?",
        "answer_tr": "Q²M HeavyCut kullanmanızı öneririz.",
    },
    "360000962217": {
        "question_tr": "Q²M Prep çizgi bıraktı - Neden?",
        "answer_tr": "Çok fazla ürün kullanmış olabilirsiniz - az miktarda tekrar uygulayın ve silin.",
    },
    "360000961917": {
        "question_tr": "Q²M Bathe+ boyama veya kaplamama zarar verir/değiştirir mi?",
        "answer_tr": "Hayır",
    },
    "360000961457": {
        "question_tr": "Q²M Bathe boyama veya kaplamama zarar verir/değiştirir mi?",
        "answer_tr": "Hayır",
    },
    "360000972358": {
        "question_tr": "Q²M Tar REDEFINED'ı şampuanımla karıştırabilir miyim?",
        "answer_tr": "Hayır",
    },
    "360000972318": {
        "question_tr": "Q²M Iron REDEFINED'ı şampuanımla karıştırabilir miyim?",
        "answer_tr": "Hayır.",
    },
    "360000972098": {
        "question_tr": "Q²M ClayBars boyama veya kaplamama zarar verir/değiştirir mi?",
        "answer_tr": "Bu ürün hafif çizilmeler bırakır.",
    },
    "360000959917": {
        "question_tr": "Q²M WaterSpot boyama veya kaplamama zarar verir/değiştirir mi?",
        "answer_tr": "Bu çözelti karoser için güvenlidir, ancak bazı kaplamalara zarar verebilir.",
    },
    "360000959797": {
        "question_tr": "Q²M Bug & Grime'ı şampuanımla karıştırabilir miyim?",
        "answer_tr": "Hayır",
    },
    "360000959697": {
        "question_tr": "Q²M Foam boyama veya kaplamama zarar verir/değiştirir mi?",
        "answer_tr": "Hayır.",
    },
    "360000971518": {
        "question_tr": "Doğru uygulanmazsa zarar verir mi?",
        "answer_tr": "Hayır",
    },
    "360000971238": {
        "question_tr": "Üzerine ek bir kaplama uygulayabilir miyim?",
        "answer_tr": "Evet, Q² Rim kullanmanızı öneririz.",
    },
    "360000945718": {
        "question_tr": "Birden fazla kat uygulayabilir miyim?",
        "answer_tr": "Evet",
    },
    "360000936517": {
        "question_tr": "Ürünün kuruması ne kadar sürer?",
        "answer_tr": "Uygulamadan sonra temizleyiciden kalan kalıntıları mutlaka alın. Kuruma süresi gerekmez.",
    },
    "360000934097": {
        "question_tr": "Ne kadar süre dayanır?",
        "answer_tr": "Q²M TireExpress en az 2 yıkama dayanır.",
    },
    "360000942438": {
        "question_tr": "Ürün kullanıldıktan sonra nasıl saklanır? Açıldıktan sonra raf ömrü nedir?",
        "answer_tr": "Dispenseri kapalı pozisyonda sabitleyin. Kuru, karanlık ve serin bir yerde saklayın. Donmasına izin vermeyin ve güneş ışığına maruz bırakmayın. Açıldıktan sonra \"kullanıma uygun\" raf ömrü 3 yıldır.",
    },
    "360000933777": {
        "question_tr": "Rotary mı DA mı?",
        "answer_tr": "Eksantrik/dual-action polisaj makinesi kullanmanızı öneririz.",
    },
    "18583770569756": {
        "question_tr": "Ürün kullanıldıktan sonra nasıl saklanır? Açıldıktan sonra raf ömrü nedir?",
        "answer_tr": "Dispenseri kapalı pozisyonda sabitleyin. Kuru, karanlık ve serin bir yerde saklayın. Donmasına izin vermeyin ve güneş ışığına maruz bırakmayın. Açıldıktan sonra \"kullanıma uygun\" raf ömrü 3 yıldır.",
    },
    "18583126898844": {
        "question_tr": "Ürün kullanıldıktan sonra nasıl saklanır? Açıldıktan sonra raf ömrü nedir?",
        "answer_tr": "Dispenseri kapalı pozisyonda sabitleyin. Kuru, karanlık ve serin bir yerde saklayın. Donmasına izin vermeyin ve güneş ışığına maruz bırakmayın. Açıldıktan sonra \"kullanıma uygun\" raf ömrü 3 yıldır.",
    },
    "18582080913692": {
        "question_tr": "Ürün kullanıldıktan sonra nasıl saklanır? Açıldıktan sonra raf ömrü nedir?",
        "answer_tr": "Dispenseri kapalı pozisyonda sabitleyin. Kuru, karanlık ve serin bir yerde saklayın. Donmasına izin vermeyin ve güneş ışığına maruz bırakmayın. Açıldıktan sonra \"kullanıma uygun\" raf ömrü 3 yıldır.",
    },
    "18581339700508": {
        "question_tr": "Q²M DeFrost'u arabaya ait olmayan camlarda kullanabilir miyim?",
        "answer_tr": "Q²M DeFrost'u aracınızın her cam yüzeyine uygulamanızı öneririz.",
    },
    "18579514490524": {
        "question_tr": "Q²M Glass+ boyama veya kaplamama zarar verir/değiştirir mi?",
        "answer_tr": "Q²M Glass+ Q² AntiFog'u kaldırır.",
    },
    "18565686602268": {
        "question_tr": "Q²M ECOWash boyama veya kaplamama zarar verir/değiştirir mi?",
        "answer_tr": "Hayır.",
    },
    "18564547262364": {
        "question_tr": "Q²M TotalRemover'ı şampuanımla karıştırabilir miyim?",
        "answer_tr": "Hayır.",
    },
    "5742812676113": {
        "question_tr": "Ürün kullanıldıktan sonra nasıl saklanır? Açıldıktan sonra raf ömrü nedir?",
        "answer_tr": "Dispenseri kapalı pozisyonda sabitleyin. Kuru, karanlık ve serin bir yerde saklayın. Donmasına izin vermeyin ve güneş ışığına maruz bırakmayın. Açıldıktan sonra \"kullanıma uygun\" raf ömrü 3 yıldır.",
    },
    "5742578780945": {
        "question_tr": "Q²M Iron WheelCleaner REDEFINED püskürttüm ama mor renk göremiyorum - Neden?",
        "answer_tr": "Aracınızda metal kontaminasyonu yoksa çözelti reaksiyona girmez ve mor renge dönmez.",
    },
    "5741123972497": {
        "question_tr": "Ürün kullanıldıktan sonra nasıl saklanır? Açıldıktan sonra raf ömrü nedir?",
        "answer_tr": "Dispenseri kapalı pozisyonda sabitleyin. Kuru, karanlık ve serin bir yerde saklayın. Donmasına izin vermeyin ve güneş ışığına maruz bırakmayın. Açıldıktan sonra \"kullanıma uygun\" raf ömrü 3 yıldır.",
    },
    "360004018537": {
        "question_tr": "Ürün kullanıldıktan sonra nasıl saklanır? Açıldıktan sonra raf ömrü nedir?",
        "answer_tr": "Dispenseri kapalı pozisyonda sabitleyin. Kuru, karanlık ve serin bir yerde saklayın. Donmasına izin vermeyin ve güneş ışığına maruz bırakmayın. Açıldıktan sonra \"kullanıma uygun\" raf ömrü 3 yıldır.",
    },
    "360004017957": {
        "question_tr": "Bu ürünle kaplanmış bir araç nasıl bakım görür?",
        "answer_tr": "Normal yıkama rutininizde Q²M Bathe+ ve Q²M Wetcoat kullanmanızı öneririz. Kaplama, pH 4 ile 9 arasındaki kimyasallara dayanacak şekilde tasarlanmıştır.",
    },
    "360000992138": {
        "question_tr": "Q²M Glass boyama veya kaplamama zarar verir/değiştirir mi?",
        "answer_tr": "Q²M Glass Q² AntiFog'u kaldırır",
    },
    "360000991898": {
        "question_tr": "Ne kadar süre dayanır?",
        "answer_tr": "Q²M Cure REDEFINED 6 haftaya kadar dayanır.",
    },
    "360000982297": {
        "question_tr": "Bu ürünle kaplanmış bir araç nasıl bakım görür?",
        "answer_tr": "Normal yıkama rutininizde Q²M Bathe+ ve Q²M Wetcoat kullanmanızı öneririz. Kaplama, pH 4 ile 9 arasındaki kimyasallara dayanacak şekilde tasarlanmıştır.",
    },
    "360000991418": {
        "question_tr": "Q²M WetCoat Essence uyguladım ve arabamda yüksek noktalar ve yağlı bir his var, neden?",
        "answer_tr": "Çok fazla ürün kullanımı yüksek noktalara yol açar - ilgili paneli tekrar yıkayıp yeniden uygulayın.",
    },
    "360000991298": {
        "question_tr": "Q²M WetCoat uyguladım ve arabamda yüksek noktalar ve yağlı bir his var, neden?",
        "answer_tr": "Çok fazla ürün kullanmak yüksek noktalara yol açar - ilgili paneli tekrar yıkayıp yeniden uygulayın.",
    },
    "360000981557": {
        "question_tr": "Polisaj kalıntısı nasıl giderilir?",
        "answer_tr": "Q²M Baldwipe/Polishwipe ve Q²M Prep spreyi veya güvenli bir IPA seyreltmesi yardımıyla.",
    },
}
```

## Script Outline (`/tmp/translate_chunk5.py`)

```python
import csv
from pathlib import Path

INPUT = "/Users/projectx/Desktop/Claude Code Projects/Products Jsons/botpress/detailagent/output/staging/chunks/chunk_5.csv"
OUTPUT = "/Users/projectx/Desktop/Claude Code Projects/Products Jsons/botpress/detailagent/output/staging/translations/chunk_5_tr.csv"

TRANSLATIONS = { ... }  # as above

def main():
    Path(OUTPUT).parent.mkdir(parents=True, exist_ok=True)
    with open(INPUT, newline="", encoding="utf-8") as f:
        rows = list(csv.DictReader(f))
    out_fields = ["target_sku","section_name","category_name","question_tr","answer_tr","question_en","answer_en","url","article_id"]
    missing = []
    with open(OUTPUT, "w", newline="", encoding="utf-8") as f:
        w = csv.DictWriter(f, fieldnames=out_fields, quoting=csv.QUOTE_MINIMAL)
        w.writeheader()
        for r in rows:
            aid = r["article_id"].strip()
            tr = TRANSLATIONS.get(aid)
            if not tr:
                missing.append(aid)
                continue
            w.writerow({
                "target_sku": r["target_sku"],
                "section_name": r["section_name"],
                "category_name": r["category_name"],
                "question_tr": tr["question_tr"],
                "answer_tr": tr["answer_tr"],
                "question_en": r["question_en"],
                "answer_en": r["answer_en"],
                "url": r["url"],
                "article_id": aid,
            })
    print(f"rows={len(rows)} written={len(rows)-len(missing)} missing={missing}")

if __name__ == "__main__":
    main()
```

## Verification Checklist

- 110 article_id keys in TRANSLATIONS — confirmed.
- All Q² product names preserved with superscript ².
- Embedded newlines preserved on `18565739075100`.
- pH/seyreltme oranları/sayılar verbatim.
- Output dir `output/staging/translations/` will be created if missing.

## Awaiting User Confirmation

Per active plan mode, I will not write code, create the CSV, or run the script until the user exits plan mode.
