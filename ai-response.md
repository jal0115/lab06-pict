Энэ даалгаврыг Category-Partition аргаар гүйцэтгэхдээ оролтын параметрүүдийг ангилж (category), ангилал бүрийн боломжит сонголтуудыг (choices) гаргана. Дараа нь сонголтуудад төлөөлөх утга оноож, хоорондын хязгаарлалт (constraints)-ыг тодорхойлоод Microsoft PICT-ийн `.model` файл бичнэ.

# 1. Category-Partition шинжилгээ

Өгөгдсөн API:

```
POST /transfer
{
  "from": ID,
  "to": ID,
  "amount": NUM,
  "currency": CODE
}
```

## 1.1. Параметр ба сонголтууд

| Параметр   | Category                   | Сонголт (Choice)    | Төлөөлөх утга                         |
| ---------- | -------------------------- | ------------------- | ------------------------------------- |
| `from`     | Данс байгаа эсэх           | FROM_EXISTS         | `ACC001`                              |
|            |                            | FROM_MISSING        | `ACC999` (байхгүй)                    |
|            | Дансны төлөв               | FROM_ACTIVE         | `frozen=false`                        |
|            |                            | FROM_FROZEN         | `frozen=true`                         |
| `to`       | Данс байгаа эсэх           | TO_EXISTS           | `ACC002`                              |
|            |                            | TO_MISSING          | `ACC999` (байхгүй)                    |
|            | Ижил данс эсэх             | TO_DIFFERENT        | `ACC002`                              |
|            |                            | TO_SAME             | `ACC001`                              |
| `amount`   | Эерэг эсэх                 | AMOUNT_POSITIVE     | `100`                                 |
|            |                            | AMOUNT_ZERO         | `0`                                   |
|            |                            | AMOUNT_NEGATIVE     | `-100`                                |
|            | Үлдэгдэл хүрэлцэх эсэх     | AMOUNT_SUFFICIENT   | Хөрвүүлсний дараах дүн ≤ үлдэгдэл     |
|            |                            | AMOUNT_INSUFFICIENT | Хөрвүүлсний дараах дүн > үлдэгдэл     |
|            | Өдрийн лимит хүрэлцэх эсэх | AMOUNT_WITHIN_LIMIT | Хөрвүүлсний дараах дүн ≤ үлдсэн лимит |
|            |                            | AMOUNT_OVER_LIMIT   | Хөрвүүлсний дараах дүн > үлдсэн лимит |
| `currency` | Валютын код                | CURRENCY_VALID      | `MNT`, `USD`                          |
|            |                            | CURRENCY_INVALID    | `XXX`                                 |
|            | Дансны валюттай харьцуулах | CURRENCY_SAME       | Дансны валюттай адил                  |
|            |                            | CURRENCY_DIFFERENT  | Дансны валютаас өөр, ханш хөрвүүлнэ   |

Анхаарах зүйл: `ACC999`-ийг байхгүй дансны төлөөлөх утга болгон ашиглаж байгаа нь жишээ юм. Туршилтын орчинд энэ ID үнэхээр байхгүй гэдгийг баталгаажуулах шаардлагатай.

## 1.2. Төлөөлөх утгын нөхцөл

Жишээлбэл, `from` дансны валютыг MNT, үлдэгдлийг 500,000 MNT, өдрийн үлдсэн лимитийг 300,000 MNT гэж үзье.

| Нөхцөл                 | Оролтын утга                    | Хүлээгдэж буй үр дүн                  |
| ---------------------- | ------------------------------- | ------------------------------------- |
| Амжилттай шилжүүлэг    | `amount=100000`, `currency=MNT` | `201 {"result":"OK"}`                 |
| Эхний данс байхгүй     | `from=ACC999`                   | `200 {"result":"ERROR_NO_FROM"}`      |
| Эхний данс царцаалттай | `from.frozen=true`              | `200 {"result":"ERROR_FROZEN"}`       |
| Хүлээн авагч байхгүй   | `to=ACC999`                     | `200 {"result":"ERROR_NO_TO"}`        |
| Нэг данс руу шилжүүлэх | `from=ACC001`, `to=ACC001`      | `200 {"result":"ERROR_SAME_ACCOUNT"}` |
| Дүн тэг                | `amount=0`                      | `200 {"result":"ERROR_BAD_AMOUNT"}`   |
| Дүн сөрөг              | `amount=-100`                   | `200 {"result":"ERROR_BAD_AMOUNT"}`   |
| Үлдэгдэл хүрэлцэхгүй   | Хөрвүүлсэн дүн 600,000 MNT      | `200 {"result":"ERROR_INSUFFICIENT"}` |
| Лимит хэтрэх           | Хөрвүүлсэн дүн 350,000 MNT      | `200 {"result":"ERROR_LIMIT"}`        |
| Өөр валют              | `currency=USD`                  | Ханшийн дагуу хөрвүүлж шалгана        |

Энд `amount`-ийн утга нь заасан `currency` валютаар илэрхийлэгдэнэ. Үлдэгдэл болон лимитийг тухайн дүнг дансны тооцооны валют руу хөрвүүлсний дараа шалгана.

# 2. Хязгаарлалт (Constraints)

Туршилтын үед нэг удаад үндсэндээ нэг алдааны нөхцөлийг шалгаж, бусад нөхцөлийг зөв байхаар тохируулна. Алдааны эрэмбийг даалгаварт өгсөн дарааллаар гэж таамаглан үзэв. Бодит API өөр дараалалтай бол хүлээгдэж буй үр дүнг түүнд нийцүүлэх шаардлагатай.

1. `from` байхгүй бол `ERROR_NO_FROM`.
2. `from` царцаалттай бол `ERROR_FROZEN`.
3. `to` байхгүй бол `ERROR_NO_TO`.
4. `from == to` бол `ERROR_SAME_ACCOUNT`.
5. `amount <= 0` бол `ERROR_BAD_AMOUNT`.
6. Үлдэгдэл хүрэлцэхгүй бол `ERROR_INSUFFICIENT`.
7. Өдрийн лимит хэтэрвэл `ERROR_LIMIT`.
8. Бүх нөхцөл зөв бол `201 {"result":"OK"}`.

## 3. Microsoft PICT модель файл

Доорх агуулгыг `transfer.model` нэртэй файлаар хадгална.

```

FromExists: YES, NO
FromStatus: ACTIVE, FROZEN, NA
ToExists: YES, NO, NA
SameAccount: SAME, DIFFERENT, NA
AmountSign: POSITIVE, ZERO, NEGATIVE, NA
BalanceCheck: SUFFICIENT, INSUFFICIENT, NA
DailyLimit: WITHIN, EXCEEDED, NA
Currency: SAME, DIFFERENT, NA

IF [FromExists] = NO THEN [FromStatus] = NA;
IF [FromExists] = NO THEN [ToExists] = NA;
IF [FromExists] = NO THEN [SameAccount] = NA;
IF [FromExists] = NO THEN [AmountSign] = NA;
IF [FromExists] = NO THEN [BalanceCheck] = NA;
IF [FromExists] = NO THEN [DailyLimit] = NA;
IF [FromExists] = NO THEN [Currency] = NA;

IF [FromStatus] = FROZEN THEN [ToExists] = NA;
IF [FromStatus] = FROZEN THEN [SameAccount] = NA;
IF [FromStatus] = FROZEN THEN [AmountSign] = NA;
IF [FromStatus] = FROZEN THEN [BalanceCheck] = NA;
IF [FromStatus] = FROZEN THEN [DailyLimit] = NA;
IF [FromStatus] = FROZEN THEN [Currency] = NA;

IF [FromStatus] = NA THEN [ToExists] = NA;

IF [ToExists] = NO THEN [SameAccount] = NA;
IF [ToExists] = NO THEN [AmountSign] = NA;
IF [ToExists] = NO THEN [BalanceCheck] = NA;
IF [ToExists] = NO THEN [DailyLimit] = NA;
IF [ToExists] = NO THEN [Currency] = NA;

IF [SameAccount] = SAME THEN [AmountSign] = NA;
IF [SameAccount] = SAME THEN [BalanceCheck] = NA;
IF [SameAccount] = SAME THEN [DailyLimit] = NA;
IF [SameAccount] = SAME THEN [Currency] = NA;

IF [AmountSign] = ZERO THEN [BalanceCheck] = NA;
IF [AmountSign] = ZERO THEN [DailyLimit] = NA;
IF [AmountSign] = ZERO THEN [Currency] = NA;

IF [AmountSign] = NEGATIVE THEN [BalanceCheck] = NA;
IF [AmountSign] = NEGATIVE THEN [DailyLimit] = NA;
IF [AmountSign] = NEGATIVE THEN [Currency] = NA;

IF [BalanceCheck] = INSUFFICIENT THEN [DailyLimit] = NA;
IF [BalanceCheck] = INSUFFICIENT THEN [Currency] = NA;

IF [DailyLimit] = EXCEEDED THEN [Currency] = NA;

```

Тайлбар: `NA` нь тухайн параметрийн сонголт одоогийн тестэд хамаарахгүй гэсэн утгатай. `Currency` параметрт зөвхөн хүчинтэй валютын кодыг (`SAME`, `DIFFERENT` гэсэн хийсвэр төлөвөөр) загварчилсан. Бодит `MNT`, `USD` кодыг тусад нь турших бол нэмэлт параметр болгон өргөтгөж болно.

## 4. PICT ашиглан тест үүсгэх

1. Кодыг `transfer.model` файлд хадгална.
2. Microsoft PICT суулгасан хавтаснаас Command Prompt нээнэ.
3. Дараах командыг ажиллуулна:

```
pict transfer.model
```

Үр дүнд PICT боломжит сонголтуудаас хослолын хамрах хүрээг хангах тестийн мөрүүдийг үүсгэнэ. Эдгээр нь шууд ажиллуулах бэлэн HTTP хүсэлт биш, харин тестийн өгөгдлийн загвар юм.

## 5. Хүлээгдэж буй үр дүнг холбох

| Тестийн нөхцөл                     | HTTP статус | `result`             |
| ---------------------------------- | ----------- | -------------------- |
| Бүх нөхцөл зөв                     | 201         | `OK`                 |
| `FromExists=NO`                    | 200         | `ERROR_NO_FROM`      |
| `FromStatus=FROZEN`                | 200         | `ERROR_FROZEN`       |
| `ToExists=NO`                      | 200         | `ERROR_NO_TO`        |
| `SameAccount=SAME`                 | 200         | `ERROR_SAME_ACCOUNT` |
| `AmountSign=ZERO` эсвэл `NEGATIVE` | 200         | `ERROR_BAD_AMOUNT`   |
| `BalanceCheck=INSUFFICIENT`        | 200         | `ERROR_INSUFFICIENT` |
| `DailyLimit=EXCEEDED`              | 200         | `ERROR_LIMIT`        |

Чухал: `INSUFFICIENT` болон `EXCEEDED` сонголтыг зөвхөн эерэг дүнтэй, өмнөх шалгалтууд амжилттай болсон тохиолдолд хэрэглэнэ. Валют өөр үед хөрвүүлсний дараах дүнгээр үлдэгдэл, лимитийг шалгах ёстой.

PICT нь сонголтуудын хослолыг багасган тест үүсгэдэг боловч дансны бодит төлөв, ханш, үлдэгдэл болон лимитийн утгыг өөрөө мэдэхгүй. Тиймээс эдгээрийг тестийн орчинд тохируулж, үүсгэсэн мөр бүрийг бодит оролтын утга болон хүлээгдэж буй хариутай холбох хэрэгтэй.
