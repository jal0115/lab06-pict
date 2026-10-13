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
