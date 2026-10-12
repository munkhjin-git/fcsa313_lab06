# Лаб 6 — Category-Partition ба PICT
Оюутан: Л.Мөнхжин B242270094

## Хэсэг A: Шилжүүлгийн функц

### Давуу эрэмбэ (олон алдаа зэрэг байвал)
Эрэмбэ: from → to → amount. Үндэслэл: to-г "from-оос өөр эсэх" гэж шалгахын тулд from-ыг эхлээд тогтоох ёстой, amount-ийн шалгалт (үлдэгдэл, лимит) нь from дансны мэдээлэлд суурилдаг тул хамгийн сүүлд хийгдэнэ.

### A1. Сонголт ба төлөөлөх утгууд

| Параметр | Сонголт | Төлөөлөх утга |
|---|---|---|
| from | Valid | байгаа, царцаагдаагүй данс (үлдэгдэл 100,000) |
| | Frozen | байгаа боловч царцаагдсан данс |
| | Missing | системд байхгүй ID (жишээ 999999) |
| to | Valid | байгаа, from-оос өөр данс |
| | Missing | байхгүй ID |
| | SameAsFrom | from-той ижил ID |
| amount | Normal | 0 < amount < үлдэгдэл, лимитийн дотор (50,000) |
| | ExactBalance | amount = үлдэгдэл (100,000), хязгаарын утга |
| | Zero | 0 (≤ 0 ангийн төлөөлөгч) |
| | OverBalance | үлдэгдлээс 1-ээр их (100,001) |
| | OverLimit | үлдэгдэл хүрэлцээтэй, гэхдээ өдрийн лимитээс их |
| currency | Same | дансны валюттай ижил |
| | Different | өөр валют, ханшийн хөрвүүлэлттэй |

Далд орчин: from-ийн үлдэгдэл, өдрийн лимит, өнөөдөр ашигласан хэмжээ. Үүнийг amount-ийн утгуудад шингээсэн (OverLimit-д үлдэгдлийг 1,000,000, лимитийг 500,000, amount-ыг 600,000 болгож бэлдэнэ). Different валютад үлдэгдэл, лимитийг хөрвүүлсний дараах дүнгээр шалгана.

### A2. Хязгаарлалтууд

| Утга | Тэмдэглэгээ | Үндэслэл |
|---|---|---|
| from=Valid | property HasFrom | данс байгаа тул үлдэгдэлтэй |
| from=Frozen | [ERROR], property HasFrom | ERROR_FROZEN буцаана, тусдаа анги |
| from=Missing | [ERROR] | ERROR_NO_FROM, үлдэгдэл байхгүй тул бусад утгатай хослуулах утгагүй |
| to=Missing | [ERROR] | ERROR_NO_TO |
| to=SameAsFrom | [ERROR], [IF HasFrom] | ERROR_SAME_ACCOUNT; from байхгүй бол "ижил" гэдэг утгагүй |
| amount=Zero | [ERROR] | ERROR_BAD_AMOUNT |
| amount=OverBalance | [ERROR], [IF HasFrom] | ERROR_INSUFFICIENT; үлдэгдэл байхгүй бол хэтрэх ойлголт байхгүй |
| amount=OverLimit | [ERROR], [IF HasFrom] | ERROR_LIMIT |
| amount=ExactBalance | [SINGLE], [IF HasFrom] | хязгаарын утга, нэг удаа шалгахад хангалттай |
| бусад | — | энгийн утга, хослоно |

Тооцоо:
- Бүрэн олонлог: 3 × 3 × 5 × 2 = **90**
- Зөвхөн [ERROR]-той: 7 алдааны утга тус бүр 1 удаа, үлдсэн нь Valid × Valid × {Normal, ExactBalance} × {Same, Different} = 4, нийт 7 + 4 = **11**
- [SINGLE] нэмбэл: 7 + 1 + 2 (happy path) = **10**
- Дүгнэлт: **90 → 10**

### A3. Спецификацийн хүснэгт

| # | from | to | amount | currency | Хүлээгдэх үр дүн |
|---|---|---|---|---|---|
| 1 | Valid | Valid | Normal | Same | 201 OK |
| 2 | Valid | Valid | Normal | Different | 201 OK |
| 3 | Valid | Valid | ExactBalance [SINGLE] | Same | 201 OK |
| 4 | Frozen | Valid | Normal | Same | ERROR_FROZEN |
| 5 | Missing | Valid | Normal | Same | ERROR_NO_FROM |
| 6 | Valid | Missing | Normal | Same | ERROR_NO_TO |
| 7 | Valid | SameAsFrom | Normal | Same | ERROR_SAME_ACCOUNT |
| 8 | Valid | Valid | Zero | Same | ERROR_BAD_AMOUNT |
| 9 | Valid | Valid | OverBalance | Same | ERROR_INSUFFICIENT |
| 10 | Valid | Valid | OverLimit | Same | ERROR_LIMIT |
| 11 | Frozen | SameAsFrom | Normal | Same | ERROR_FROZEN (давхар нөхцөл, from эхэлж шалгагдана) |
| 12 | Valid | Missing | Zero | Same | ERROR_NO_TO (давхар нөхцөл, to нь amount-аас өмнө) |


## Хэсэг B1: Браузерын жишээ (PICT)

| Хувилбар | Мөрийн тоо | Файл |
|---|---|---|
| Бүрэн олонлог (3×3×2×2×2×2) | 144 | — |
| 2-way (pairwise) | 10 | results/browser-2way.txt |
| 3-way (`/o:3`) | 20 | results/browser-3way.txt |

**2-way: 10 мөр, гараар хийсэн 9-өөс нэг илүү.** Лекц дээр гараар 9 мөрөнд
багтаасан бөгөөд 9 нь онолын доод хязгаар юм, учир нь Content × Cookies-ийн
3 × 3 = 9 хос тус бүр өөр мөр шаардана. PICT 10 мөр өгсөн нь greedy
алгоритмтай холбоотой. PICT мөр бүрийг сонгохдоо тухайн агшинд хамгийн олон
хараахан хамрагдаагүй хосыг хамарсан мөрийг авдаг. Алхам бүрийн ийм сайн
сонголт нийт хамгийн цөөн мөрийг баталгаажуулдаггүй тул хамгийн цөөнийг
ЗААВАЛ өгөхгүй, харин бүх хосыг хамарсан, хангалттай цөөн олонлог өгдөг.
Мөн би гараар хийхдээ "-" (хамаагүй) нүд ашигласан бол PICT нүд бүрд
тодорхой утга бөглөдөг.

**3-way: 20 мөр.** Бүрэн олонлог 144-ийн ойролцоогоор 14 хувь (7 дахин цөөн).
Гэхдээ 2-way-ээс хоёр дахин олон мөр байна, учир нь бүх гурвал (triple)
хамрагдах ёстой. t өсөх тусам тестийн тоо өсдөг боловч бүрэн олонлогоос
хамаагүй бага хэвээр байна. Эрсдэл өндөртэй системд 3-way, түүнээс дээш
хамралт, бусад тохиолдолд 2-way хэрэглэнэ.

## Хэсэг B2: Шилжүүлгийн model (PICT)

| Хувилбар | Мөр | Happy path мөр | Нэг мөрөнд алдаа |
|---|---|---|---|
| transfer-nc (хязгаарлалтгүй, 90-ийн pairwise) | 15 | 0 | 1: 5 · 2: 6 · 3: 4 |
| transfer (A2-ын IF хязгаарлалттай) | 17 | 0 | 1: 9 · 2: 4 · 3: 4 |
| transfer-neg (~ сөрөг утгатай) | 18 | 4 | 0: 4 · 1: 14 |

- **transfer-nc, transfer:** happy path мөр огт гараагүй. PICT алдааны утгуудыг
  хоорондоо хосолсон тул нэг мөрөнд 3 хүртэл алдаа орсон. Эхний алдаа бусдыгаа
  нууна (error masking), иймээс ийм мөр алдаа тус бүрийг тусад нь шалгаж
  чадахгүй. Давуу эрэмбэ (from → to → amount) тодорхойгүй бол хүлээгдэх үр дүн
  ч тодорхойгүй.
- **transfer-nc → transfer:** хязгаарлалт нэмэхэд мөр 15 → 17 болж **өссөн**.
  Хязгаарлалт нь утгагүй хослолуудыг хасдаг ч үлдсэн хосуудыг бүгдийг нь
  хамрахын тулд PICT нэмэлт мөр шаардсан.
- **transfer-neg:** нэг мөрөнд хамгийн ихдээ 1 алдаа, мөн 4 happy path мөр
  гарсан. Мөр бүр яг нэг алдааны ангийг шалгана, тиймээс oracle тодорхой.
- **~ хэрэглэхэд A2-ын хязгаарлалт хэрэгтэй хэвээр үү?** Хэсэгчлэн. from=Missing
  үед эерэг утга ExactBalance үлдэгдэл шаардах тул утгагүй хослол үүсдэг, иймээс
  `IF [From] = "Missing" THEN [Amount] <> "ExactBalance"` хэвээр хэрэгтэй.
  Харин SameAsFrom, OverBalance, OverLimit нь өөрсдөө сөрөг утга тул
  ~Missing-тэй нэг мөрөнд орж чадахгүй, тэдгээрийн хязгаарлалт хэрэггүй болсон.

### Бодит тест кейс (setup + оролт + oracle)

Setup-д нийтлэг: ханш 1 USD = 3,400 MNT; ACC-1 (MNT, үлдэгдэл 1,000,000,
өдрийн лимит 500,000, өнөөдөр ашигласан 0); ACC-2 (MNT, идэвхтэй);
ACC-F (MNT, царцаагдсан); ACC-LOW (MNT, үлдэгдэл 100,000, лимит 500,000).

| # | Эх мөр | Setup | Оролт | Oracle |
|---|---|---|---|---|
| 1 | transfer-neg: `Valid, Valid, Normal, Different` (happy path) | ACC-1, ACC-2 | `{"from":"ACC-1","to":"ACC-2","amount":100,"currency":"USD"}` | 201, `{"result":"OK"}` (100 USD = 340,000 MNT, үлдэгдэл ба лимит хэтрээгүй) |
| 2 | transfer-neg: `~Frozen, Valid, Normal, Same` | ACC-F, ACC-2 | `{"from":"ACC-F","to":"ACC-2","amount":50000,"currency":"MNT"}` | 200, `{"result":"ERROR_FROZEN"}` |
| 3 | transfer-neg: `Valid, Valid, ~OverBalance, Different` | ACC-LOW, ACC-2 | `{"from":"ACC-LOW","to":"ACC-2","amount":50,"currency":"USD"}` | 200, `{"result":"ERROR_INSUFFICIENT"}` (50 USD = 170,000 MNT > 100,000, хөрвүүлсний дараах дүнгээр шалгана) |
| 4 | transfer: `Frozen, SameAsFrom, Zero, Same` (олон алдаатай) | ACC-F | `{"from":"ACC-F","to":"ACC-F","amount":0,"currency":"MNT"}` | 200, `{"result":"ERROR_FROZEN"}`: давуу эрэмбэ from → to → amount тул from-ийн алдаа эхэлж буцна, ERROR_SAME_ACCOUNT, ERROR_BAD_AMOUNT-ыг нууна |

### Model файлууд ба мөрийн тооны тайлбар

Дээрх мөрийн тоо толгой мөрийг оруулаагүй: `tail -n +2 results/<файл> | wc -l`.
Model файлд зөвхөн латин тэмдэгт ашигласан (кирилл comment байвал PICT
хязгаарлалтыг үл тооно).

`browser.model`: Content, Cookies (Allow, Restrict, Block); Popups, Addons, Attacks, Forgeries (Yes, No).

`transfer.model` (`transfer-nc.model` нь хязгаарлалтгүй хувилбар):

```
From:     Valid, Frozen, Missing
To:       Valid, Missing, SameAsFrom
Amount:   Normal, ExactBalance, Zero, OverBalance, OverLimit
Currency: Same, Different

IF [From] = "Missing" THEN [Amount] IN {"Normal", "Zero"};
IF [From] = "Missing" THEN [To] <> "SameAsFrom";
```

`transfer-neg.model`:

```
From:     Valid, ~Frozen, ~Missing
To:       Valid, ~Missing, ~SameAsFrom
Amount:   Normal, ExactBalance, ~Zero, ~OverBalance, ~OverLimit
Currency: Same, Different

IF [From] = "Missing" THEN [Amount] <> "ExactBalance";
```

## Дүгнэлт

Гараар хийсэн category-partition (A3) нь алдааны семантикийг илүү сайн барьсан: ERROR анги бүр тусдаа мөртэй, SINGLE захын тохиолдол ба давхар нөхцөлийн мөр (11, 12) давуу эрэмбээр (from → to → amount) тодорхой үр дүнтэй болсон. PICT нь харин хосын хамралтыг баталгаажуулдаг: transfer-nc-ийн 15 мөр бүх хосыг хамарсан гэдгийг PICT өөрөө шалгадаг, гараар сонгосон 12 мөрөнд ийм баталгаа байхгүй. Гэвч хязгаарлалтгүй PICT happy path мөр огт өгөөгүй бөгөөд нэг мөрөнд 3 хүртэл алдаа хольсон тул error masking-ийн улмаас алдаа тус бүрийг тусад нь шалгаж чадахгүй. `~` сөрөг утга үүнийг засаж (нэг мөрөнд ≤1 алдаа, 4 happy path мөр) category-partition-ийн [ERROR] тэмдэглэгээтэй дүйсэн. Хязгаарлалт нэмэхэд мөр 15 → 17 болж өссөн нь хязгаарлалт ба хосын хамралт хоорондоо нөлөөлдгийг харуулсан. PICT нь утгагүй хослолыг өөрөө таньдаггүй бөгөөд хүлээгдэх үр дүнг (oracle) ч өгдөггүй тул from=Missing үед ExactBalance-ийг хасах хязгаарлалтыг би домэйны мэдлэгээр бичсэн. Иймээс хоёр арга бие биеийг орлохгүй: category-partition нь юуг, яагаад шалгахыг шийддэг бол PICT нь тэдгээрийн хослолыг цөөн мөрөөр хамруулдаг. Хамгийн их бодол шаардсан алхам нь олон алдаа зэрэг байх үеийн давуу эрэмбийг тогтоох явдал байлаа, учир нь тодорхойлолт үүнийг бичээгүй тул A3-ын 11, 12-р мөр болон олон алдаатай мөрийн oracle бүгд энэ шийдвэрээс хамаарсан. Хоёр дахь нь `~`-ийн дараа A2-ын аль хязгаарлалт хэвээр хэрэгтэйг тодорхойлох байсан: SameAsFrom, OverBalance зэрэг өөрсдөө сөрөг болсон тул хязгаарлалт шаардлагагүй болсон ч эерэг ExactBalance-ийг from=Missing-тэй хослуулахгүй байх нэг хязгаарлалт үлдсэн.
