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
