# LLM Search

This project demonstrates a method to enhance search queries using large language models (LLMs). Although it might be costly for large websites (estimated at $1.5 per thousand searches), the implementation requires just a few days without the need for fine-tuning models, vectorization, schema changes, or additional servers.

### Why Is This Needed?

Internal searches on websites are often poor, slow, and rely heavily on keyword indexing. For example, to enable someone to find "a garden apartment with up to 5 rooms in Tel-Aviv" all this text needs to be indexed. Most tables consist of keys rather than text. While tables have indexes, someone needs to choose them, and to get an appropriate page, you need to click on multiple checkboxes and dropdown lists. It can become a headache on large classified, e-commerce and/or travel sites, and limits the interaction between the user and the site (for example, if they want to expand or narrow the search).

### The Solution

The method proposed here converts a search query (of any length, including spelling mistakes) from unstructured free text to a JSON structure tailored to each site. You can use the JSON output to construct an SQL/Elastic query running on the regular database or convert it to a filtered URL to be used across the site.

### How It Works

1. Create a hierarchical schema of all filter fields on the site (no special description, just how the keys and values are named on the site).
2. Send this schema along with the original user query to any completion API. The smaller, the better. I used OpenAI's gpt-4o-mini which is $0.15/1M for input and $0.6/1M for output tokens
3. Receive a structure of all the filters found, all keywords that didn't match any filters, as well as synonyms for the keywords (to expand the textual search).

### Example Interaction

See here a [sample interaction](https://x.com/mluggy/status/1833152820024840199) with [yad2.co.il](https://www.yad2.co.il) site. It took me a few hours to map all the real estate and vehicle categories myself (for the site owner, this would take a few minutes).

### JSON Schema Example

```json
{
  "category": [
    {
      "id": "realestate",
      "name": "נדל\"ן",
      "subcategory": [
        { "id": "forsale", "name": "למכירה" },
        { "id": "rent", "name": "להשכרה" },
        {
          "id": "commercial",
          "name": "נדל\"ן מסחרי",
          "dealType": [
            { "id": 0, "name": "מכירה" },
            { "id": 1, "name": "השכרה" }
          ],
          "property": [
            { "id": 12, "name": "משרדים" },
            { "id": 13, "name": "חנויות/ שטח מסחרי" },
            { "id": 62, "name": "חלל עבודה משותף" },
            { "id": 20, "name": "אולמות" },
            { "id": 19, "name": "מבני תעשיה" },
            { "id": 16, "name": "מחסנים" },
            { "id": 18, "name": "מגרשים" },
            { "id": 35, "name": "בניין משרדים" },
            { "id": 38, "name": "חניון" },
            { "id": 58, "name": "מרתף" },
            { "id": 24, "name": "כללי" },
            { "id": 59, "name": "סטודיו" },
            { "id": 14, "name": "קליניקות" },
            { "id": 60, "name": "בית מלון" }
          ],
          "air-conditioner": { "id": 1, "name": "מיזוג" },
          "cooling-room": { "id": 1, "name": "חדר קירור" },
          "kitchenette": { "id": 1, "name": "מטבחון" },
          "high-ceiling": { "id": 1, "name": "תקרה גבוהה" },
          "alarm": { "id": 1, "name": "אזעקה" },
          "meeting-room": { "id": 1, "name": "חדר ישיבות" },
          "cameras": { "id": 1, "name": "מצלמות" },
          "communication-room": { "id": 1, "name": "תקשורת" },
          "loading-ramp": { "id": 1, "name": "רמפת העמסה" },
          "airConditioner": false,
          "accessibility": false,
          "bars": false,
          "furniture": false,
          "renovated": false,
          "asset_exclusive": false,
          "propertyCondition": false,
          "toilet": [
            { "id": 1, "name": "שירותים בבניין" },
            { "id": 2, "name": "שירותים בנכס" }
          ],
          "warhouse": [
            { "id": 1, "name": "מחסן בבניין" },
            { "id": 2, "name": "מחסן בנכס" }
          ],
          "shelter": [
            { "id": 1, "name": "ממ״ד בבניין" },
            { "id": 2, "name": "ממ״ד בנכס" }
          ]
        }
      ],
      "propertyGroup": [
        {
          "id": "apartments",
          "name": "דירות",
          "type": [
            { "id": 1, "name": "דירה" },
            { "id": 3, "name": "דירת גן" },
            { "id": 6, "name": "גג/ פנטהאוז" },
            { "id": 7, "name": "דופלקס" },
            { "id": 25, "name": "תיירות ונופש" },
            { "id": 49, "name": "מרתף/ פרטר" },
            { "id": 51, "name": "טריפלקס" },
            { "id": 11, "name": "יחידת דיור" },
            { "id": 4, "name": "סטודיו/ לופט" }
          ]
        },
        {
          "id": "houses",
          "name": "בתים",
          "type": [
            { "id": 5, "name": "בית פרטי/ קוטג'" },
            { "id": 39, "name": "דו משפחתי" },
            { "id": 32, "name": "משק חקלאי/ נחלה" },
            { "id": 55, "name": "משק עזר" }
          ]
        },
        {
          "id": "misc",
          "name": "אחר",
          "type": [
            { "id": 33, "name": "מגרשים" },
            { "id": 61, "name": "דיור מוגן" },
            { "id": 44, "name": "בניין מגורים" },
            { "id": 45, "name": "מחסן" },
            { "id": 30, "name": "חניה" },
            { "id": 50, "name": "קב' רכישה/ זכות לנכס" },
            { "id": 41, "name": "כללי" }
          ]
        }
      ],
      "price": { "min": 0, "max": 20000000 },
      "rooms": [
        { "id": "1", "name": "חדר אחד" },
        { "id": "1.5", "name": "חדר וחצי" },
        { "id": "2", "name": "שני חדרים" },
        { "id": "2.5", "name": "שניים וחצי חדרים" },
        { "id": "3", "name": "שלושה חדרים" },
        { "id": "3.5", "name": "שלושה וחצי חדרים" },
        { "id": "4", "name": "ארבעה חדרים" },
        { "id": "4.5", "name": "ארבעה וחצי חדרים" },
        { "id": "5", "name": "חמש חדרים" },
        { "id": "5.5", "name": "חמש וחצי חדרים" },
        { "id": "6", "name": "ששה חדרים ומעלה" }
      ],
      "imageOnly": { "id": 1, "name": "עם תמונה" },
      "priceOnly": { "id": 1, "name": "עם מחיר" },
      "moshavimKibutzim": { "id": 1, "name": "רק מושבים וקיבוצים" },
      "priceDropped": { "id": 1, "name": "נכסים שמחירם ירד" },
      "brokerage": { "id": 1, "name": "תיווך" },
      "newFromContractor": { "id": 1, "name": "יזם/קבלן" },
      "parking": { "id": 1, "name": "חניה" },
      "elevator": { "id": 1, "name": "מעלית" },
      "airConditioner": { "id": 1, "name": "מיזוג" },
      "balcony": { "id": 1, "name": "מרפסת" },
      "shelter": { "id": 1, "name": "ממ״ד" },
      "bars": { "id": 1, "name": "סורגים" },
      "asset_exclusive": { "id": 1, "name": "בבלעדיות" },
      "furniture": { "id": 1, "name": "מרוהטת" },
      "handicap": { "id": 1, "name": "גישה לנכים" },
      "renovated": { "id": 1, "name": "משופצת" },
      "accessibility": { "id": 1, "name": "גישה לנכים" },
      "warhouse": { "id": 1, "name": "מחסן" },
      "propertyCondition": [
        { "id": 1, "name": "חדש מקבלן (לא גרו בכלל)" },
        { "id": 6, "name": "חדש (נכס בן עד 10 שנים)" },
        { "id": 2, "name": "משופץ (שופץ ב-5 שנים האחרונות)" },
        { "id": 3, "name": "במצב שמור (במצב טוב, לא שופץ)" },
        { "id": 5, "name": "דרוש שיפוץ (זקוק לעבודת שיפוץ)" }
      ],
      "floor": [
        { "id": -1, "name": "מרתף" },
        { "id": 0, "name": "קרקע" },
        { "id": 1, "name": "קומה ראשונה" },
        { "id": 2, "name": "קומה שנייה" },
        { "id": 3, "name": "קומה שלישית" },
        { "id": 4, "name": "קומה רביעית" },
        { "id": 5, "name": "קומה חמישית" },
        { "id": 6, "name": "קומה שישית" },
        { "id": 7, "name": "קומה שביעית" },
        { "id": 8, "name": "קומה שמינית" },
        { "id": 9, "name": "קומה תשיעית" },
        { "id": 10, "name": "קומה עשירית" },
        { "id": 11, "name": "קומה אחת עשרה" },
        { "id": 12, "name": "קומה שתיים עשרה" },
        { "id": 12, "name": "קומה שלוש עשרה" },
        { "id": 13, "name": "קומה ארבע עשרה" },
        { "id": 14, "name": "קומה חמש עשרה" },
        { "id": 15, "name": "קומה שש עשרה" },
        { "id": 16, "name": "קומה שבע עשרה" },
        { "id": 17, "name": "קומה שמונה עשרה" },
        { "id": 18, "name": "קומה תשע עשרה" },
        { "id": 19, "name": "קומה עשרים ומעלה" }
      ],
      "squaremeter": { "min": 0, "max": 10000 },
      "EnterDate": [{ "id": -1, "name": "כניסה מיידית" }]
    },
    {
      "id": "vehicles",
      "name": "רכב",
      "subcategory": [
        {
          "id": "cars",
          "name": "פרטיים ומסחריים",
          "carFamilyType": [
            { "id": 10, "name": "קרוסאוברים" },
            { "id": 2, "name": "משפחתיים" },
            { "id": 8, "name": "יוקרה" },
            { "id": 5, "name": "ג׳יפים" },
            { "id": 1, "name": "קטנים" },
            { "id": 9, "name": "מיניוואנים" },
            { "id": 3, "name": "מנהלים" },
            { "id": 4, "name": "ספורט" },
            { "id": 7, "name": "מסחריים" },
            { "id": 6, "name": "טנדרים" }
          ],
          "manufacturer": [
            { "id": "baw", "name": "BAW" },
            { "id": "denza", "name": "Denza/דנזה" },
            { "id": "eveasy", "name": "EVEASY" },
            { "id": "ram", "name": "RAM/ראם" },
            { "id": "290", "name": "XPENG" },
            { "id": "333", "name": "ZEEKR" },
            { "id": "1", "name": "אאודי" },
            { "id": "abarth", "name": "אבארט" },
            { "id": "autobianchi", "name": "אוטוביאנקי" },
            { "id": "oldsmobile", "name": "אולדסמוביל" },
            { "id": "austin", "name": "אוסטין" },
            { "id": "opel", "name": "אופל" },
            { "id": "ora", "name": "אורה / Ora" },
            { "id": "aiways", "name": "איווייס" },
            { "id": "iveco", "name": "איווקו" },
            { "id": "infiniti", "name": "אינפיניטי" },
            { "id": "isuzu", "name": "איסוזו" },
            { "id": "levc", "name": "אל.אי.וי.סי / LEVC" },
            { "id": "lti", "name": "אל.טי.איי" },
            { "id": "alfaromeo", "name": "אלפא רומיאו" },
            { "id": "alpine", "name": "אלפין / ALPINE" },
            { "id": "mg", "name": "אם. ג'י. / MG" },
            { "id": "astonmartin", "name": "אסטון מרטין" },
            { "id": "few", "name": "אף. אי. דאבליו / FEW" },
            { "id": "bmw", "name": "ב.מ.וו" },
            { "id": "byd", "name": "בי.ווי.די / BYD" },
            { "id": "buick", "name": "ביואיק" },
            { "id": "bentley", "name": "בנטלי" },
            { "id": "jaecoo", "name": "ג'אקו/Jaecoo" },
            { "id": "gac", "name": "ג'י.איי.סי/ GAC" },
            { "id": "gmc", "name": "ג'י.אם.סי / GMC" },
            { "id": "geo", "name": "ג'יאו / Geo" },
            { "id": "jiayuan", "name": "ג'יאיוואן/ Jiayuan" },
            { "id": "jac", "name": "ג'יי.איי.סי / JAC" },
            { "id": "geely", "name": "ג'ילי - Geely" },
            { "id": "jeep", "name": "ג'יפ / Jeep" },
            { "id": "jeep-taa", "name": "ג'יפ תע''ר" },
            { "id": "genesis", "name": "ג'נסיס" },
            { "id": "greatwall", "name": "גרייט וול" },
            { "id": "dacia", "name": "דאצ'יה" },
            { "id": "dodge", "name": "דודג'" },
            { "id": "dongfeng", "name": "דונגפנג" },
            { "id": "ds", "name": "די.אס / DS" },
            { "id": "daewoo", "name": "דייהו" },
            { "id": "daihatsu", "name": "דייהטסו" },
            { "id": "hummer", "name": "האמר" },
            { "id": "hongqi", "name": "הונגצ'י / HONGQI" },
            { "id": "honda", "name": "הונדה" },
            { "id": "hino", "name": "הינו HINO" },
            { "id": "wey", "name": "ווי / WEY" },
            { "id": "voyah", "name": "וויה / VOYAH" },
            { "id": "volvo", "name": "וולוו" },
            { "id": "tata", "name": "טאטא" },
            { "id": "19", "name": "טויוטה" },
            { "id": "62", "name": "טסלה" },
            { "id": "jaguar", "name": "יגואר" },
            { "id": "hyundai", "name": "יונדאי" },
            { "id": "lada", "name": "לאדה" },
            { "id": "lynkco", "name": "לינק&קו / Lynk&Co" },
            { "id": "lincoln", "name": "לינקולן" },
            { "id": "leapmotor", "name": "ליפמוטור / leapmotor" },
            { "id": "lichi", "name": "ליצ'י" },
            { "id": "lamborghini", "name": "למבורגיני" },
            { "id": "landrover", "name": "לנד רובר" },
            { "id": "lancia", "name": "לנצ'יה" },
            { "id": "lexus", "name": "לקסוס" },
            { "id": "27", "name": "מאזדה" },
            { "id": "man", "name": "מאן" },
            { "id": "maserati", "name": "מזראטי" },
            { "id": "mini", "name": "מיני" },
            { "id": "mitsubishi", "name": "מיצובישי" },
            { "id": "mclaren", "name": "מקלארן / McLaren" },
            { "id": "maxus", "name": "מקסוס" },
            { "id": "mercedes", "name": "מרצדס" },
            { "id": "nio", "name": "ניאו / NIO" },
            { "id": "nissan", "name": "ניסאן" },
            { "id": "nanjing", "name": "ננג'ינג" },
            { "id": "saab", "name": "סאאב" },
            { "id": "sunliving", "name": "סאן ליוינג / Sun Living" },
            { "id": "ssangyong", "name": "סאנגיונג" },
            { "id": "sunshine", "name": "סאנשיין" },
            { "id": "subaru", "name": "סובארו" },
            { "id": "36", "name": "סוזוקי" },
            { "id": "seat", "name": "סיאט" },
            { "id": "citroen", "name": "סיטרואן" },
            { "id": "smart", "name": "סמארט" },
            { "id": "centro", "name": "סנטרו" },
            { "id": "skoda", "name": "סקודה" },
            { "id": "skywell", "name": "סקייוול" },
            { "id": "seres", "name": "סרס / SERES" },
            { "id": "polestar", "name": "פולסטאר / POLESTAR" },
            { "id": "volkswagen", "name": "פולקסווגן" },
            { "id": "pontiac", "name": "פונטיאק" },
            { "id": "ford", "name": "פורד" },
            { "id": "porsche", "name": "פורשה" },
            { "id": "forthing", "name": "פורתינג / FORTHING" },
            { "id": "piaggio", "name": "פיאג'ו" },
            { "id": "fiat", "name": "פיאט" },
            { "id": "peugeot", "name": "פיג'ו" },
            { "id": "ferrari", "name": "פרארי" },
            { "id": "chery", "name": "צ'רי / Chery" },
            { "id": "cadillac", "name": "קאדילאק" },
            { "id": "karma", "name": "קארמה/ Karma" },
            { "id": "cupra", "name": "קופרה" },
            { "id": "48", "name": "קיה" },
            { "id": "chrysler", "name": "קרייזלר" },
            { "id": "rover", "name": "רובר" },
            { "id": "rollsroyce", "name": "רולס רויס / Rolls Royse" },
            { "id": "renault", "name": "רנו" },
            { "id": "chevrolet", "name": "שברולט" }
          ],
          "engineval": { "min": 800, "max": 6800 },
          "seats": { "min": 2, "max": 11 },
          "engineType": [
            { "id": 9, "name": "גט'ד / בנזין" },
            { "id": 8, "name": "היברידי חשמל / דיזל" },
            { "id": 7, "name": "חשמלי" },
            { "id": 6, "name": "גט'ד" },
            { "id": 5, "name": "היברידי חשמל / בנזין" },
            { "id": 4, "name": "גפ'ם / בנזין" },
            { "id": 3, "name": "טורבו דיזל" },
            { "id": 2, "name": "דיזל" },
            { "id": 1, "name": "בנזין" }
          ],
          "gearBox": [
            { "id": 0, "name": "ידנית" },
            { "id": 1, "name": "אוטומטית" },
            { "id": 6, "name": "טיפטרוניק" },
            { "id": 9, "name": "רובוטית" }
          ],
          "group_color": [
            { "id": 5, "name": "אדום" },
            { "id": 7, "name": "אפור" },
            { "id": 4, "name": "ורוד" },
            { "id": 6, "name": "חום" },
            { "id": 2, "name": "ירוק" },
            { "id": 9, "name": "כחול" },
            { "id": 3, "name": "כתום" },
            { "id": 10, "name": "לבן" },
            { "id": 8, "name": "סגול" },
            { "id": 11, "name": "צהוב" },
            { "id": 1, "name": "שחור" }
          ],
          "is_trade_in_button": { "id": 1, "name": "זמין בטרייד אין" },
          "is_payment_installments": {
            "id": 1,
            "name": "זמין בעסקת מימון"
          },
          "disabledFriendly": { "id": 1, "name": "מותאם לנכים" }
        },
        {
          "id": "motorcycles",
          "name": "אופנועים",
          "manufacturer": [
            { "id": "ajp", "name": "AJP" },
            { "id": "bsa", "name": "BSA" },
            { "id": "bse", "name": "BSE" },
            { "id": "bultaco", "name": "Bultaco" },
            { "id": "cfmoto", "name": "CF MOTO" },
            { "id": "changjiang", "name": "CHANGJIANG" },
            { "id": "hm-moto-italia", "name": "HM מוטו איטליה" },
            { "id": "horwin", "name": "Horwin" },
            { "id": "ktm", "name": "KTM" },
            { "id": "mv-agusta", "name": "MV Agusta" },
            { "id": "mz", "name": "MZ" },
            { "id": "piaggio", "name": "Piaggio" },
            { "id": "qj-motor", "name": "QJ MOTOR" },
            { "id": "swm", "name": "SWM" },
            { "id": "tiger", "name": "TIGER" },
            { "id": "tm-racing", "name": "TM Racing" },
            { "id": "um", "name": "UM" },
            { "id": "voge", "name": "Voge" },
            { "id": "zap", "name": "ZAP" },
            { "id": "aust", "name": "אוסט" },
            { "id": "ural", "name": "אורל/Ural" },
            { "id": "italjet", "name": "איטלגט" },
            { "id": "indian", "name": "אינדיאן" },
            { "id": "energica", "name": "אנרג'יקה" },
            { "id": "aprilia", "name": "אפריליה" },
            { "id": "bmw", "name": "ב.מ.וו" },
            { "id": "beta", "name": "בטא" },
            { "id": "buell", "name": "ביואל" },
            { "id": "benelli", "name": "בנלי" },
            { "id": "jawa", "name": "ג'אווה" },
            { "id": "gasgas", "name": "גאס-גאס" },
            { "id": "golden-motorsik", "name": "גולדן מוטורסיק" },
            { "id": "ducati", "name": "דוקאטי" },
            { "id": "daelim", "name": "דיאלים" },
            { "id": "derbi", "name": "דרבי" },
            { "id": "havale", "name": "הוואלה" },
            { "id": "honda", "name": "הונדה" },
            { "id": "husaberg", "name": "הוסאברג" },
            { "id": "husqvarna", "name": "הוסקוורנה" },
            { "id": "harley-davidson", "name": "הרלי דיווידסון" },
            { "id": "zongshen", "name": "זונגשן" },
            { "id": "zontes", "name": "זונטס" },
            { "id": "zero-motorcycles", "name": "זירו- אופנועים חשמליים" },
            { "id": "taro", "name": "טארו" },
            { "id": "triumph", "name": "טריומף" },
            { "id": "java", "name": "יאווה" },
            { "id": "hyosung", "name": "יוסאנג" },
            { "id": "yamaha", "name": "ימאהה" },
            { "id": "loncin", "name": "לונגיה" },
            { "id": "lifan", "name": "ליפאן" },
            { "id": "lambretta", "name": "למברטה" },
            { "id": "motoguzzi", "name": "מוטוגוצי" },
            { "id": "motomorini", "name": "מוטומוריני" },
            { "id": "mondial", "name": "מונדיאל" },
            { "id": "minibike", "name": "מיני בייק" },
            { "id": "norton", "name": "נורטון" },
            { "id": "sym", "name": "סאן יאנג" },
            { "id": "suzuki", "name": "סוזוקי" },
            { "id": "sur-ron", "name": "סורון" },
            { "id": "stomp", "name": "סטומפ" },
            { "id": "scorpa", "name": "סקורפה" },
            { "id": "skyteam", "name": "סקיי-טים" },
            { "id": "fantic", "name": "פאנטיק" },
            { "id": "piaggio-rikshaw", "name": "פיאגו ריקשה" },
            { "id": "peugeot", "name": "פיג'ו" },
            { "id": "ktm", "name": "ק.ט.מ" },
            { "id": "cagiva", "name": "קאג'יבה" },
            { "id": "kawasaki", "name": "קאוואסאקי" },
            { "id": "kwang-yang", "name": "קואנג יאנג" },
            { "id": "kuberg", "name": "קוברג" },
            { "id": "keeway", "name": "קיוואי" },
            { "id": "kymco", "name": "קימקו" },
            { "id": "cleveland", "name": "קליבלנד" },
            { "id": "xingyue", "name": "קסינג-יו" },
            { "id": "royal-enfield", "name": "רויאל אנפילד" },
            { "id": "riggio", "name": "ריגיו" },
            { "id": "sherco", "name": "שרקו" },
            { "id": "other", "name": "אחר" }
          ],
          "license": [
            { "id": 4, "name": "ללא הגבלה A" },
            { "id": 3, "name": "A1 עד 47 כ\"ס" },
            { "id": 2, "name": "A2 עד 125 סמ\"ק/14 כ\"ס" }
          ],
          "engineval": { "min": 100, "max": 1700 }
        },
        {
          "id": "scooters",
          "name": "קטנועים",
          "manufacturer": [
            { "id": "cpi", "name": "CPI" },
            { "id": "evt", "name": "EVT" },
            { "id": "fym", "name": "FYM" },
            { "id": "gmi", "name": "GMI" },
            { "id": "gogoro", "name": "Gogoro / גוגורו" },
            { "id": "lml-italia", "name": "LML Italia" },
            { "id": "niu-nqi", "name": "NIU NQI" },
            { "id": "pgo", "name": "PGO" },
            { "id": "qj-motor", "name": "QJ MOTOR" },
            { "id": "tgb", "name": "TGB" },
            { "id": "um", "name": "UM" },
            { "id": "voge", "name": "VOGE" },
            { "id": "yadea", "name": "YADEA" },
            { "id": "zontes", "name": "ZONTES" },
            { "id": "e-max", "name": "אי-מקס - קטנועים חשמליים" },
            { "id": "aprilia", "name": "אפריליה" },
            { "id": "bmw", "name": "ב.מ.וו" },
            { "id": "beta", "name": "בטה" },
            {
              "id": "blitz-motors",
              "name": "בליץ מוטורס - קטנועים חשמליים"
            },
            { "id": "benelli", "name": "בנלי" },
            { "id": "gilera", "name": "ג'ילרה" },
            { "id": "dofren", "name": "דופרן" },
            { "id": "daelim", "name": "דיאלים" },
            { "id": "di-blasi", "name": "דיבלאסי" },
            { "id": "derbi", "name": "דרבי" },
            { "id": "honda", "name": "הונדה" },
            { "id": "vectrix", "name": "וקטריקס" },
            { "id": "zap-scooters", "name": "זאפ - קטנועים חשמליים" },
            { "id": "zongshen", "name": "זונגשן" },
            { "id": "zenen", "name": "זנן" },
            { "id": "taro", "name": "טארו" },
            { "id": "hyosung", "name": "יוסאנג" },
            { "id": "yamaha", "name": "ימאהה" },
            { "id": "loncin", "name": "לונגיה" },
            { "id": "lifan", "name": "לונסין" },
            { "id": "lifan", "name": "ליפאן" },
            { "id": "lambretta", "name": "למברטה" },
            { "id": "mbk", "name": "מ.ב.ק" },
            { "id": "nipponia", "name": "ניפוניה" },
            { "id": "sym", "name": "סאן יאנג" },
            { "id": "suzuki", "name": "סוזוקי" },
            { "id": "silence", "name": "סיילנס - קטנועים חשמליים" },
            { "id": "piaggio", "name": "פיאג'ו" },
            { "id": "peugeot", "name": "פיג'ו" },
            { "id": "cagiva", "name": "קאג'יבה" },
            { "id": "kawasaki", "name": "קאוואסאקי" },
            { "id": "quadro", "name": "קוואדרו" },
            { "id": "kwang-yang", "name": "קוואנג-יאנג" },
            { "id": "keeway", "name": "קיוואי" },
            { "id": "kymco", "name": "קימקו" },
            { "id": "cleveland", "name": "קליבלנד" },
            { "id": "xingyue", "name": "קסינג-יו" },
            { "id": "other", "name": "אחר" }
          ],
          "license": [
            { "id": 4, "name": "ללא הגבלה A" },
            { "id": 3, "name": "A1 עד 47 כ\"ס" },
            { "id": 2, "name": "A2 עד 125 סמ\"ק/14 כ\"ס" }
          ],
          "engineval": { "min": 50, "max": 850 }
        }
      ],
      "year": { "min": 1976, "max": 2024 },
      "price": { "min": 0, "max": 200000 },
      "km": { "min": 0, "max": 70000 },
      "hand": [
        { "id": 0, "name": "0 קילומטר" },
        { "id": 1, "name": "יד ראשונה" },
        { "id": 2, "name": "יד שנייה" },
        { "id": 3, "name": "יד שלישית" },
        { "id": 4, "name": "יד רביעית" },
        { "id": 5, "name": "יד חמישית ומעלה" }
      ],
      "imgOnly": { "id": 1, "name": "עם תמונה" },
      "priceOnly": { "id": 1, "name": "עם מחיר" },
      "isCommercial": { "id": 1, "name": "מודעות מסוכנויות" }
    }
  ],
  "city": [
    { "id": 3000, "name": "ירושלים" },
    { "id": 5000, "name": "תל אביב-יפו" },
    { "id": 4000, "name": "חיפה" },
    { "id": 6400, "name": "הרצליה" },
    { "id": 6900, "name": "כפר סבא" },
    { "id": 8700, "name": "רעננה" },
    { "id": 2600, "name": "אילת" },
    { "id": 7900, "name": "פתח תקווה" },
    { "id": 8300, "name": "ראשון לציון" },
    { "id": 6100, "name": "בני ברק" },
    { "id": 1200, "name": "מודיעין מכבים רעות" }
  ]
}
```

### System Prompt Example

```
You are an AI assistant designed to interpret user unstructured search queries into structured JSON using a custom JSON schema.

1. Select the single best category that captures the user's intent

2. Select the single best subcategory within that category

3. Identify as many relevant params from the top level, category and/or subcategory level ignoring params marked as false

4. Output identified params using their key (i.e "category", "subcategory", "property") and the most appropriate "id" from the JSON schema as value (i.e "category": "realestate", "shelter": "1", "group_color": 7, "property": 12). These are "single-value-params"

5. If user asked for multiple values of the same param, output them as array of ids (i.e "property": [12, 20], "license": [4, 3, 2). These are "multiple-value-params". If this param has min/max, do your best to accomodate (i.e "price": {"max": 700000} or "price": {"min": 10000, "max": 25000}). These are "range-value-params"

6. If user negated one or more values and there are other values remaining for that param, select all other values (i.e "not tiptronic" would output: "gearBox": [0, 1, 9])

7. Output any keywords from the user's query that weren't included and/or matched as a category, subcategory or any other params (i.e "red toyota rav4 from a docter" would output keywords": ["rav4", "doctor", "nurse"] because red and toyota were matched as group_color and manufacturer)

8. Output up to 3 synonyms of the remaining keywords (i.e "synonyms": ["synonym1", "synonym2", "synonym3"])

9. If user message enclosed inside <sql> tag generate a valid query:

SELECT * FROM category.subcategory
WHERE single-value-param = "id" AND multiple-value-param IN ["value1", "value2"] AND range-value-param >= min-value AND range-value-param < max-value
AND (keywords LIKE '%keyword1%' OR keywords LIKE '%keyword2%' OR keywords LIKE '%synonym1%' ...)

10. If user message enclosed inside <url> tag, generate a valid link by using the params identified in the previous steps (param key as query param key and param value as query param value). For price and rooms, always pass range (i.e "below 10000" is "price=0-10000" and "5 rooms" is "rooms=5-5")
https://www.yad2.co.il/{category}/{subcategory}?single-value-param=value&multiple-value-param=value1,value2&range-value-param=minvalue-maxvalue&text=keywords

11. If user message enclosed inside <elastic> tag, generate a valid elasticsearch DSL

12. Do not generate any code comments or syntax hints
```

### License

This project is licensed under the MIT License - see the [LICENSE](https://github.com/mluggy/llm-search/blob/main/LICENSE) file for details.

### Acknowledgments

Special thanks to Yad2 for providing a platform to test this search enhancement method.

Happy searching! ❤️

Michael Lugassy

Follow me on [LinkedIn](https://www.linkedin.com/in/mluggy/) & [X/Twitter.](https://x.com/mluggy)
