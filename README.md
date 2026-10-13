# Лаб 6: Category-Partition ба PICT

**Оюутан:** Г. Лувсанжал · **Код:** B242270016
**Хичээл:** F.CSA313 Программ хангамжийн чанарын баталгаа ба тест (2026)

## Хэсэг A: Category-Partition (POST /transfer)

**Давуу эрэмбэ:** from → to → amount → currency. Данс байхгүй, царцсан бол үлдэгдэл, лимитийг шалгах утгагүй тул эхлээд from, дараа нь to, хамгийн сүүлд amount-ийг шалгана. Amount дотор BAD_AMOUNT → INSUFFICIENT → LIMIT гэж эрэмбэлнэ.

### A1. Сонголт ба төлөөлөх утгууд

| Параметр | Төлөөлөх утгууд |
|---|---|
| from | Valid (байгаа, царцаагүй), Frozen (царцсан), Missing (байхгүй) |
| to | Valid (байгаа, from-оос өөр), Missing (байхгүй), SameAsFrom (from-той ижил) |
| amount | Normal (0 < amount, үлдэгдэл ба лимитийн дотор), ExactBalance (үлдэгдэлтэй тэнцүү), Zero (0), OverBalance (үлдэгдлээс их), OverLimit (өдрийн лимитээс их) |
| currency | Same (дансны валюттай ижил), Different (өөр, ханшийн хөрвүүлэлттэй) |
| Далд орчин | Өдрийн лимит хэр дүүрсэн байсан бэ. OverLimit-ийг үүсгэхийн тулд setup-д лимитийг тогтоох ёстой. |

### A2. Хязгаарлалтууд

| Утга | Тэмдэглэгээ | Үндэслэл |
|---|---|---|
| from=Frozen | [ERROR] | ERROR_FROZEN буцаана, бусад параметрийг шалгахгүй. |
| from=Missing | [ERROR] | ERROR_NO_FROM буцаана. |
| to=Missing | [ERROR] | ERROR_NO_TO буцаана. |
| to=SameAsFrom | [ERROR] | ERROR_SAME_ACCOUNT буцаана. |
| amount=Zero | [ERROR] | amount ≤ 0 бол ERROR_BAD_AMOUNT. |
| amount=OverBalance | [ERROR] | ERROR_INSUFFICIENT. |
| amount=OverLimit | [ERROR] | ERROR_LIMIT. |
| amount=ExactBalance | [SINGLE] | Хил хязгаарын нэг тохиолдол, нэг удаа шалгахад хангалттай, амжилттай дуусна. |
| from=Missing | [IF] amount=Normal | Байхгүй дансны үлдэгдэл гэж байхгүй тул amount утгагүй. |
| to=Missing | [IF] amount=Normal | to алдаа эхэлж гарсан тул amount шалгагдахгүй, утгыг тогтмол авна. |
| to=SameAsFrom | [IF] from=Valid | Байхгүй/царцсан данстай "ижил данс" гэдэг ялгаагүй, ойлгомжгүй. |

**Тооцоо:** бүрэн олонлог 3 × 3 × 5 × 2 = **90** → [IF] хязгаарлалтын дараа **38** → [ERROR]/[SINGLE]-ийн дараа **10** (2 happy path + 1 SINGLE + 7 ERROR).

### A3. Спецификацийн хүснэгт

| # | from | to | amount | currency | Хүлээгдэх үр дүн |
|---|---|---|---|---|---|
| 1 | Valid | Valid | Normal | Same | 201 OK (happy path) |
| 2 | Valid | Valid | Normal | Different | 201 OK (хөрвүүлэлттэй) |
| 3 | Valid | Valid | ExactBalance | Same | 201 OK [SINGLE] |
| 4 | Missing | Valid | Normal | Same | 200 ERROR_NO_FROM |
| 5 | Frozen | Valid | Normal | Same | 200 ERROR_FROZEN |
| 6 | Valid | Missing | Normal | Same | 200 ERROR_NO_TO |
| 7 | Valid | SameAsFrom | Normal | Same | 200 ERROR_SAME_ACCOUNT |
| 8 | Valid | Valid | Zero | Same | 200 ERROR_BAD_AMOUNT |
| 9 | Valid | Valid | OverBalance | Same | 200 ERROR_INSUFFICIENT |
| 10 | Valid | Valid | OverLimit | Same | 200 ERROR_LIMIT |
| 11 | Valid | Valid | OverBalance | Different | 200 ERROR_INSUFFICIENT (хөрвүүлсний дараах дүнгээр) |
| 12 | Frozen | Missing | Normal | Same | 200 ERROR_FROZEN (давхар алдаа: from-ийг эхэлж шалгана) |
| 13 | Missing | Missing | Normal | Same | 200 ERROR_NO_FROM (давхар алдаа: from эхэлнэ) |
| 14 | Valid | Valid | үлдэгдэл ба лимит хоёулангаас их | Same | 200 ERROR_INSUFFICIENT (INSUFFICIENT нь LIMIT-ээс өмнө) |

## Хэсэг B: PICT

### B1. Лекцийн browser жишээ

Model: `browser.model` (Content, Cookies, Popups, Addons, Attacks, Forgeries). Гаралт: `results/browser-2way.txt`, `results/browser-3way.txt`.

| Хувилбар | Мөрийн тоо | Бүрэн олонлог |
|---|---|---|
| 2-way (pairwise) | 10 | 144 (3 × 3 × 2⁴) |
| 3-way (/o:3) | 20 | 144 |

**2-way яагаад 9 биш 10 болов:** Content ба Cookies хоёр тус бүр 3 утгатай тул тэдний 3 × 3 = 9 хосыг хамрахад хамгийн багадаа 9 мөр хэрэгтэй. Лекц дээр гараар 9 мөрөнд багтаасан. PICT greedy алгоритмтай: мөр бүрд одоохондоо хамраагүй хосыг хамгийн их хамарсан мөрийг сонгодог тул хамгийн цөөн мөрийг ЗААВАЛ өгдөггүй. Тиймээс энэ удаад 1 мөр илүү гарсан ч бүх хос хамрагдсан хэвээр.

**3-way:** 20 мөр нь бүрэн олонлог 144-ийн ойролцоогоор 14%. Гурвалсан хослол хамруулахад 2-way-ээс 2 дахин олон мөр хэрэгтэй боловч бүрэн олонлогоос хамаагүй цөөн хэвээр.

### B2. Шилжүүлгийн model PICT-ээр

Model файлууд: `transfer-nc.model` (хязгаарлалтгүй), `transfer.model` (A2-ын [IF] хязгаарлалттай), `transfer-neg.model` ([ERROR] утгууд `~`-тэй). Гаралт: `results/transfer-nc.txt`, `results/transfer.txt`, `results/transfer-neg.txt`.

Happy path гэдэг нь from=Valid, to=Valid, amount=Normal эсвэл ExactBalance байх мөр.

| Хувилбар | Мөрийн тоо | Happy path мөр | Нэг мөр дэх хамгийн их алдаа | 2+ алдаатай мөр |
|---|---|---|---|---|
| transfer-nc (хязгаарлалтгүй) | 15 | 0 | 3 | 10 |
| transfer (IF хязгаарлалттай) | 15 | 0 | 2 | 9 |
| transfer-neg (`~` утгатай) | 18 | 4 | 1 | 0 |

**Ажиглалт:** хязгаарлалтгүй болон IF хязгаарлалттай хувилбарт happy path мөр огт гараагүй. Pairwise нь бүх ХОСЫГ хамруулдаг тул ихэнх мөрөнд хоёр, гурван алдааны утга хамт орсон. Ийм мөрөнд эхний алдаа бусдыгаа нууна (error masking), тиймээс аль алдаа буцахыг давуу эрэмбээс (from → to → amount) л тодорхойлно, өөр мөр нь тусдаа алдааг шалгахгүй. `~` ашигласан хувилбарт нэг мөрөнд хамгийн ихдээ нэг сөрөг утга орж, happy path мөр 4 гарсан, мөр бүр нэг л зүйлийг шалгадаг болсон.

**`~` хэрэглэхэд A2-ын хязгаарлалт хэрэгтэй хэвээр үү:** хэсэгчлэн. `to=Missing → amount=Normal` болон `to=SameAsFrom → from=Valid` хязгаарлалтууд илүүдэж, сөрөг утга нэг мөрөнд ганцхан байх тул давхар алдаа аяндаа гарахгүй. Харин `from=Missing → amount=Normal` хязгаарлалт хэрэгтэй хэвээр, учир нь үүнгүй бол `~Missing, Valid, ExactBalance` гэсэн мөр гарч, байхгүй дансны үлдэгдэлтэй тэнцүү дүн гэсэн утгагүй тест үүсдэг.

#### Бодит тест кейсүүд

Тест бүр өөрийн данс үүсгэнэ (бусад тестээс хамаарахгүй). Ханш: 1 EUR = 1.10 USD гэж setup-д тогтооно.

| # | PICT мөр | Setup | Оролт (POST /transfer) | Oracle |
|---|---|---|---|---|
| TC1 | transfer-neg: `Valid, Valid, Normal, Different` (happy path) | T1-A: USD, үлдэгдэл 1000, царцаагүй, өдрийн лимит 5000, ашигласан 0; T1-B: USD данс | `{"from":"T1-A","to":"T1-B","amount":100,"currency":"EUR"}` | 201, `{"result":"OK"}` (100 EUR = 110 USD ≤ 1000 ба ≤ 5000) |
| TC2 | transfer-neg: `~Frozen, Valid, Normal, Same` (нэг алдаа) | T2-A: USD, үлдэгдэл 1000, ЦАРЦАГДСАН; T2-B: USD данс | `{"from":"T2-A","to":"T2-B","amount":100,"currency":"USD"}` | 200, `{"result":"ERROR_FROZEN"}` |
| TC3 | transfer-neg: `Valid, Valid, ~OverBalance, Different` (нэг алдаа) | T3-A: USD, үлдэгдэл 1000, царцаагүй, лимит 5000, ашигласан 0; T3-B: USD данс | `{"from":"T3-A","to":"T3-B","amount":2000,"currency":"EUR"}` | 200, `{"result":"ERROR_INSUFFICIENT"}` (2000 EUR = 2200 USD > 1000, лимит 5000-аас хэтрээгүй) |
| TC4 | transfer: `Frozen, Missing, Normal, Same` (давхар алдаа) | T4-A: USD, үлдэгдэл 1000, ЦАРЦАГДСАН; `T4-X` данс үүсгэхгүй | `{"from":"T4-A","to":"T4-X","amount":100,"currency":"USD"}` | 200, `{"result":"ERROR_FROZEN"}`: давуу эрэмбээр from-ийг to-оос өмнө шалгана, ERROR_NO_TO нууцлагдана |
