# Лаборатори №6: Category-Partition ба PICT

Ч. Мөнхбаяр — B242270045  
Хичээл: F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест

Олон алдаа давхацвал `from` байгаа эсэх → `from` царцсан эсэх → `to` байгаа эсэх → ижил данс эсэх → `amount <= 0` → үлдэгдэл → өдрийн лимит гэсэн дарааллаар эхний алдааг буцаана. Шилжүүлгийг үргэлжлүүлэх боломжгүй хамгийн суурь нөхцөлөөс эхэлж шалгахын тулд энэ дарааллыг сонгосон.

## A. Category-Partition

### A1. Сонголт ба төлөөлөх утга

Суурь өгөгдөл: `A` данс 1,000,000 MNT үлдэгдэлтэй, өдрийн үлдсэн лимит 500,000 MNT; `B` нь тусдаа хүчинтэй данс. Өөр валютын жишээнд `1 USD = 3,500 MNT` ханш хэрэглэнэ. `LimitUsage` нь хүсэлтийн body-д байхгүй ч үр дүнд нөлөөлдөг далд орчны сонголт.

| Параметр | Сонголт | Төлөөлөх утга | Тэмдэглэгээ | Үндэслэл |
|---|---|---|---|---|
| `From` | `Valid` | `A` | ердийн | Данс байгаа, царцаагдаагүй. |
| `From` | `Frozen` | `F` | `[ERROR]`, `[IF Amount=Normal, Currency=Same, LimitUsage=Available]` | Бусад мөнгөний нөхцөлөөс үл хамааран `ERROR_FROZEN` гарна. `To`-г чөлөөтэй үлдээж давхар нөхцөл шалгана. |
| `From` | `Missing` | `X` | `[ERROR]`, `[SINGLE]` | Байхгүй эх данс бусад шалгалтыг утгагүй болгоно. |
| `To` | `Valid` | `B` | ердийн | Хүлээн авагч байгаа ба `from`-оос өөр. |
| `To` | `Missing` | `Y` | `[ERROR]`, `[IF Amount=Normal, Currency=Same, LimitUsage=Available]` | `ERROR_NO_TO`-г мөнгөний нэмэлт алдаагүй шалгана. |
| `To` | `SameAsFrom` | `A` | `[ERROR]`, `[IF Amount=Normal, Currency=Same, LimitUsage=Available]` | Ижил данс руу шилжүүлэхэд `ERROR_SAME_ACCOUNT` гарна. |
| `Amount` | `Zero` | `0 MNT` | `[ERROR]`, `[SINGLE]` | `amount <= 0` ангийг нэг цэвэр тестээр шалгана. |
| `Amount` | `Normal` | `100,000 MNT` эсвэл `100 USD` | ердийн | Хөрвүүлсний дараа үлдэгдэл, лимитийн дотор байна. |
| `Amount` | `OverBalance` | `1,100,000 MNT` эсвэл `400 USD` | `[ERROR]`, `[IF From=Valid, To=Valid, LimitUsage=Available]` | Хөрвүүлсний дараа үлдэгдлээс их тул `ERROR_INSUFFICIENT`. |
| `Amount` | `OverLimit` | `600,000 MNT` эсвэл `200 USD` | `[ERROR]`, `[IF From=Valid, To=Valid, LimitUsage=Available]` | Үлдэгдэл хүрэлцэх ч өдрийн үлдсэн лимитээс их тул `ERROR_LIMIT`. |
| `Currency` | `Same` | `MNT` | ердийн | Хөрвүүлэлт хийхгүй. |
| `Currency` | `Different` | `USD` | `[IF exchangeRateExists]` | Тестийн setup-д тогтмол ханш бэлдэнэ. |
| `LimitUsage` | `Available` | үлдсэн лимит `500,000 MNT` | ердийн | Хэвийн болон мөнгөний хязгаарын тестэд хэрэглэнэ. |
| `LimitUsage` | `Exhausted` | үлдсэн лимит `0 MNT` | `[ERROR]`, `[IF From=Valid, To=Valid, Amount=Normal]` | Энгийн эерэг дүн ч `ERROR_LIMIT` өгөх ёстой. |

### A2. Хязгаарлалт ба тоо

Хязгаарлалтгүй бүрэн олонлог:

`3 × 3 × 4 × 2 × 2 = 144`

`transfer.model`-д дараах дүрмүүд орсон.

- `From=Missing` болон `Amount=Zero` нь `[SINGLE]`: бусад параметр суурь хүчинтэй утгатай байна.
- `From=Frozen` үед мөнгөний сонголтуудыг `Normal/Same/Available` болгоно, харин `To`-г чөлөөтэй үлдээнэ.
- `To=Missing` эсвэл `SameAsFrom` үед мөнгөний сонголтуудыг суурь утгаар тогтооно.
- `OverBalance`, `OverLimit` үед хоёр данс хүчинтэй, лимитийн төлөв `Available` байна.
- `LimitUsage=Exhausted` үед хоёр данс хүчинтэй, дүн `Normal` байна.

Хязгаарлалтын дараа `From=Missing` 1, `From=Frozen` 3, `From=Valid` 11 боломж үлдэнэ. Иймээс category-partition-ийн логик олонлог **144 → 15** болсон. Энэ 15 нь боломжит бүтэн комбинацын тоо; доорх 13 нь PICT-ийн pairwise covering array-ийн мөрийн тоо.

### A3. Спецификацийн хүснэгт

| № | From | To | Amount | Currency | Limit setup | Хүлээгдэх үр дүн |
|---:|---|---|---:|---|---|---|
| 1 | `A` valid | `B` valid | 100,000 | MNT | 500,000 үлдсэн | `201 {"result":"OK"}` |
| 2 | `A` valid | `B` valid | 100 | USD | 500,000 үлдсэн, 1 USD=3,500 MNT | `201 {"result":"OK"}` |
| 3 | `X` missing | `B` valid | 100,000 | MNT | 500,000 үлдсэн | `200 {"result":"ERROR_NO_FROM"}` |
| 4 | `F` frozen | `B` valid | 100,000 | MNT | 500,000 үлдсэн | `200 {"result":"ERROR_FROZEN"}` |
| 5 | `A` valid | `Y` missing | 100,000 | MNT | 500,000 үлдсэн | `200 {"result":"ERROR_NO_TO"}` |
| 6 | `A` valid | `A` same | 100,000 | MNT | 500,000 үлдсэн | `200 {"result":"ERROR_SAME_ACCOUNT"}` |
| 7 | `A` valid | `B` valid | 0 | MNT | 500,000 үлдсэн | `200 {"result":"ERROR_BAD_AMOUNT"}` — `[SINGLE]` |
| 8 | `A` valid | `B` valid | 1,100,000 | MNT | 500,000 үлдсэн | `200 {"result":"ERROR_INSUFFICIENT"}` |
| 9 | `A` valid | `B` valid | 600,000 | MNT | 500,000 үлдсэн | `200 {"result":"ERROR_LIMIT"}` |
| 10 | `A` valid | `B` valid | 100,000 | MNT | 0 үлдсэн | `200 {"result":"ERROR_LIMIT"}` |
| 11 | `F` frozen | `Y` missing | 100,000 | MNT | 500,000 үлдсэн | `200 {"result":"ERROR_FROZEN"}` — давхар нөхцөл |
| 12 | `A` valid | `Y` missing | 0 | MNT | 500,000 үлдсэн | `200 {"result":"ERROR_NO_TO"}` — давхар нөхцөл |

## B. PICT pairwise covering array

PICT-ийг Ubuntu дээр эх кодоос build хийсэн нотолгоо [results/pict-build.txt](results/pict-build.txt)-д байна. Моделүүдийг seed буюу `/r` сонголтгүй ажиллуулсан.

### B1. Browser модель

| Хамралт | PICT мөр | Бүрэн олонлогтой харьцуулалт |
|---|---:|---:|
| 2-way | 10 | 144-ийн 6.9% |
| 3-way | 20 | 144-ийн 13.9% |
| Бүрэн олонлог | 144 | 100% |

Лекцийн гараар хийсэн хүснэгт 9 мөртэй байсан бол PICT [10 мөр](results/browser-2way.txt) гаргасан. PICT нь greedy алгоритмаар тухайн алхамд хамгийн олон шинэ хос хамруулах мөрийг сонгодог ч глобал хамгийн бага мөрийн тоог батлахгүй. Тиймээс 10 мөр гарсан нь pairwise хамралт дутуу гэсэн үг биш.

3-way хувилбар [20 мөр](results/browser-3way.txt) болсон. Энэ нь 2-way-гаас хоёр дахин их боловч бүтэн 144 комбинацаас нэлээд бага.

### B2. Transfer моделүүд

Энд happy path гэдэг нь `Valid / Valid / Normal / Same|Different / Available` мөр. Алдааны тоонд `Frozen`, `Missing`, `SameAsFrom`, `Zero`, `OverBalance`, `OverLimit`, `Exhausted` утгуудыг тооцсон.

| Модель | PICT мөр | Happy path | Нэг мөр дэх хамгийн их алдаа | Тайлбар |
|---|---:|---:|---:|---|
| [transfer-nc.model](transfer-nc.model) | 14 | 1 | 4 | Хязгаарлалтгүй тул error masking их. |
| [transfer.model](transfer.model) | 13 | 0 | 2 | A2 хязгаарлалтууд боломжгүй хослолыг хассан. |
| [transfer-neg.model](transfer-neg.model) | 18 | 2 | 1 | `~` нь нэг мөрөнд хамгийн ихдээ нэг сөрөг утга оруулсан. |

Хязгаарлалтгүй гаралтад жишээ нь `Missing / Missing / Zero / Different / Exhausted` гэсэн дөрвөн алдаатай мөр гарсан. Аль алдаа бусдыгаа нууж байгааг зөвхөн тогтоосон давуу эрэмбээр мэдэх тул энэ мөрийн зорилго тодорхой биш. Хязгаарлалттай модель 13 мөр болсон ч PICT pairwise хамралтыг л зорьдог учраас яг хүссэн бүхэл happy path мөрийг заавал гаргаагүй; ийм чухал мөрийг гараар нэмэх хэрэгтэй.

`~` хэрэглэснээр алдааны утгууд нэг мөрөнд давхардахгүй болсон. Манай `transfer-neg.model`-ийн эерэг утгууд бүгд хоорондоо боломжтой тул A2-ын алдаа тусгаарлах хязгаарлалтууд давхар хэрэггүй. Харин ханш байгаа эсэх, хөрвүүлсний дараах тоон харьцаа зэрэг setup-ийн бодит нөхцөлийг тест өгөгдөл дээр хэвээр барина.

#### PICT мөрөөс бодит тест кейс

**TC1 — happy path (`Valid / Valid / Normal / Different / Available`)**

- Setup: `A` данс 1,000,000 MNT, үлдсэн лимит 500,000 MNT; `B` хүчинтэй; 1 USD=3,500 MNT.
- Input: `POST /transfer {"from":"A","to":"B","amount":100,"currency":"USD"}`
- Oracle: HTTP 201, `{"result":"OK"}`; 350,000 MNT-тэй тэнцэх дүн шилжинэ.

**TC2 — байхгүй эх данс (`~Missing / Valid / Normal / Different / Available`)**

- Setup: `X` данс байхгүй; `B` хүчинтэй; ханш бэлэн.
- Input: `POST /transfer {"from":"X","to":"B","amount":100,"currency":"USD"}`
- Oracle: HTTP 200, `{"result":"ERROR_NO_FROM"}`; үлдэгдэл өөрчлөгдөхгүй.

**TC3 — лимит хэтэрсэн (`Valid / Valid / ~OverLimit / Different / Available`)**

- Setup: `A` данс 1,000,000 MNT, үлдсэн лимит 500,000 MNT; `B` хүчинтэй; 1 USD=3,500 MNT.
- Input: `POST /transfer {"from":"A","to":"B","amount":200,"currency":"USD"}`
- Oracle: 700,000 MNT нь үлдэгдэлд хүрэлцэх ч лимитээс их тул HTTP 200, `{"result":"ERROR_LIMIT"}`.

## Дүгнэлт

Category-partition нь оролтын утга бүрийн утга, алдааны oracle-ыг тодорхой бичихэд илүү тохиромжтой. `[ERROR]`, `[SINGLE]`, `[IF]` тэмдэглэгээ нь 144 комбинацыг 15 утгатай логик олонлог болгож багасгав. Гараар хийсэн хүснэгтэд happy path болон алдааны анги бүрийг зориуд оруулж чадсан. Мөн олон алдааны давуу эрэмбийг урьдчилан тогтоосноор давхар нөхцөлийн oracle тодорхой болсон. PICT нь бүх хосын хамралтыг автоматаар баталгаажуулж, гараар анзаарахад хэцүү хослолуудыг гаргасан. Гэхдээ greedy алгоритм хамгийн цөөн мөрийг батлахгүй, мөн утгын семантикийг өөрөө ойлгохгүй. Сөрөг утгыг энгийн утгаар өгөхөд error masking гарсан бол `~` тэмдэглэгээ үүнийг нэг алдаа болгон цэвэрлэсэн. Энэ ажилд хамгийн их бодол шаардсан хэсэг нь далд `LimitUsage` сонголт, валютын хөрвүүлэлт, алдааны давуу эрэмбийг нэг логик модельд нийцүүлэх байв. Иймээс практикт category-partition-оор утга ба oracle-аа тогтоогоод, PICT-ээр pairwise хамралтаа нөхөх нь илүү найдвартай.

## Ажиллуулах команд

```bash
pict browser.model | tee results/browser-2way.txt
pict browser.model /o:3 > results/browser-3way.txt
pict transfer-nc.model > results/transfer-nc.txt
pict transfer.model > results/transfer.txt
pict transfer-neg.model > results/transfer-neg.txt
```
