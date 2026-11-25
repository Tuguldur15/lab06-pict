# Лаб 6: Category-Partition ба PICT

- **Оюутны нэр:** Tuguldur
- **Код:** b242270050

---

## Хэсэг A: Category-Partition (Шилжүүлгийн функц)

### Олон алдааны давуу эрэмбэ
Нэг хүсэлтэд хэд хэдэн алдаа зэрэг байвал `from → to → amount` дарааллаар шалгаж, эхний олдсон алдааг буцаана гэж тогтоолоо.
- **Үндэслэл:** Сайн хэрэгжүүлэлт эхлээд эх дансыг (байгаа эсэх, царцсан эсэх), дараа нь хүлээн авагчийг олж, хамгийн сүүлд дүнг шалгадаг. Дүнг шалгахын тулд (үлдэгдэл, лимит) эх данс заавал хэрэгтэй тул дансны шалгалт дүнгийн шалгалтаас өмнө байх нь логик.
- **Дүн доторх дараалал:** `ERROR_BAD_AMOUNT → ERROR_INSUFFICIENT → ERROR_LIMIT`. Жишээ нь `amount` нь үлдэгдэл ба лимит хоёроос хоёулаасаа хэтэрвэл `ERROR_INSUFFICIENT` буцна.

### A1: Сонголт ба төлөөлөх утгууд

| Параметр / сонголт | Төлөөлөх утгууд |
| :--- | :--- |
| `from` (эх данс) | `Valid` (хүчинтэй), `Frozen` (царцсан), `Missing` (байхгүй) |
| `to` (хүлээн авагч) | `Valid`, `Missing`, `SameAsFrom` (`from`-той ижил) |
| `amount` (дүн) | `Zero` (0, мөн сөрөг дүнг төлөөлнө), `Normal` (үлдэгдэл, лимит дотор), `Exact` (яг үлдэгдэлтэй тэнцүү), `OverBalance` (үлдэгдлээс их), `OverLimit` (үлдэгдэл дотор боловч өдрийн лимитээс их) |
| `currency` (валют) | `Same` (дансныхтай ижил), `Different` (өөр, ханшийн хөрвүүлэлттэй) |

- **Далд орчин:** Өдрийн лимит хэр дүүрсэн байсан — **Хоосон** (өнөөдөр шилжүүлээгүй), **Лимитэд ойр** (өмнөх шилжүүлгээр дүүрсэн).
- Далд орчны сонголт нь `OverLimit`-ийн үр дүнг тодорхойлдог. Яг ижил дүн *"лимит хоосон"* үед `OK`, *"лимитэд ойр"* үед `ERROR_LIMIT` болдог. Иймд `OverLimit` тест бүрийн setup-д лимитийн төлөвийг тодорхой заана.

### A2: Хязгаарлалтууд

| Утга | Тэмдэглэгээ | Үндэслэл |
| :--- | :--- | :--- |
| `from = Frozen` | `[ERROR]` | `ERROR_FROZEN` буцаах тул бусад параметрийн шалгалтыг цааш хийхгүй, ганц удаа шалгана |
| `from = Missing` | `[ERROR]` | `ERROR_NO_FROM` буцаана, бусад параметр нь дараагийн шалгалтад хүрэхгүй |
| `to = Missing` | `[ERROR]` | `ERROR_NO_TO` буцаана |
| `to = SameAsFrom` | `[ERROR]` | `ERROR_SAME_ACCOUNT` буцаана |
| `amount = Zero` | `[ERROR]` | `ERROR_BAD_AMOUNT` буцаана (≤ 0 бүгд нэг анги) |
| `amount = OverBalance` | `[ERROR]` | `ERROR_INSUFFICIENT` буцаана |
| `amount = OverLimit` | `[ERROR]` | `ERROR_LIMIT` буцаана |
| `amount = Exact` | `[SINGLE]` | Хязгаарын (boundary) утга: яг үлдэгдлээр шилжүүлэхэд амжилттай байх ёстой. Нэг удаа шалгахад хангалттай, бусад утгатай хослуулах шаардлагагүй |
| `to = SameAsFrom` | `[IF from = Valid]` | `"from-той ижил"` гэдэг нь `from` байж байж утга авна |
| `amount = OverBalance / OverLimit / Zero` | `[IF from = Valid AND to = Valid]` | Эх данс, хүлээн авагч хүчинтэй байж л дүнг шалгадаг (давуу эрэмбийн дагуу) |
| `currency = Different` | *(хязгаарлалтгүй)* | Энгийн утга, бусад бүгдтэй хослоно, хөрвүүлсний дараа үлдэгдэл, лимитийг шалгадаг |

**N → M тооцоо:**
- **Бүрэн олонлог:** 3 (from) × 3 (to) × 5 (amount) × 2 (currency) = 90.
- **Хязгаарлалтын дараа:** `Valid-Valid-Normal × 2` валют = `2`, `[SINGLE] Exact` = `1`, `[ERROR]` утга бүр нэг удаа (`Frozen`, `Missing(from)`, `Missing(to)`, `SameAsFrom`, `Zero`, `OverBalance`, `OverLimit`) = `7`. **Нийт: 90 → 10.**

### A3: Спецификацийн хүснэгт

| № | `from` | `to` | `amount` | `currency` | Хүлээгдэх үр дүн | Тэмдэглэл |
| :-: | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | `Valid` | `Valid` | `Normal` | `Same` | `201 OK` | Happy path |
| 2 | `Valid` | `Valid` | `Normal` | `Different` | `201 OK` | Happy path + хөрвүүлэлт |
| 3 | `Valid` | `Valid` | `Exact` | `Same` | `201 OK` | `[SINGLE]` яг үлдэгдлээр |
| 4 | `Missing` | `Valid` | `Normal` | `Same` | `200 ERROR_NO_FROM` | `[ERROR]` |
| 5 | `Frozen` | `Valid` | `Normal` | `Same` | `200 ERROR_FROZEN` | `[ERROR]` |
| 6 | `Valid` | `Missing` | `Normal` | `Same` | `200 ERROR_NO_TO` | `[ERROR]` |
| 7 | `Valid` | `SameAsFrom` | `Normal` | `Same` | `200 ERROR_SAME_ACCOUNT` | `[ERROR]` |
| 8 | `Valid` | `Valid` | `Zero` | `Same` | `200 ERROR_BAD_AMOUNT` | `[ERROR]` |
| 9 | `Valid` | `Valid` | `OverBalance` | `Same` | `200 ERROR_INSUFFICIENT` | `[ERROR]` |
| 10 | `Valid` | `Valid` | `OverLimit` | `Different` | `200 ERROR_LIMIT` | `[ERROR]`, хөрвүүлсний дараах дүнгээр лимит шалгана |
| 11 | `Frozen` | `Missing` | `Normal` | `Same` | `200 ERROR_FROZEN` | Давхар нөхцөл: `from → to` эрэмбээр `ERROR_FROZEN` |
| 12 | `Valid` | `SameAsFrom` | `Zero` | `Same` | `200 ERROR_SAME_ACCOUNT` | Давхар нөхцөл: `to → amount` эрэмбээр |
| 13 | `Valid` | `Valid` | `OverBalance` (мөн лимитээс их) | `Same` | `200 ERROR_INSUFFICIENT` | Давхар нөхцөл: `INSUFFICIENT → LIMIT` эрэмбээр |

---

## Хэсэг B: PICT

PICT-ийг эх кодоос бүтээсэн, нотолгоо: `results/pict-build.txt`.

### B1: Browser model
- **Файлууд:** `browser.model`, `results/browser-2way.txt`, `results/browser-3way.txt`

| Хувилбар | Мөрийн тоо |
| :--- | :---: |
| Бүрэн олонлог (3×3×2×2×2×2) | 144 |
| 2-way | 10 |
| 3-way (`/o:3`) | 20 |

- **2-way:** 10 мөр гарав, лекц дээр гараар 9 мөрөнд багтаасан. PICT нь greedy алгоритмтай. Мөр бүрийг нэмэхдээ одоогоор хамрагдаагүй хосуудыг хамгийн их хамарсан мөрийг сонгодог боловч нийт мөрийн тоог хамгийн цөөн болгохыг баталгаажуулдаггүй. `Content × Cookies` хоёр параметр 3×3 = 9 хостой тул 9 нь доод хязгаар бөгөөд PICT-ийн гаралтад ганц хос `(Allow, Block)` давтагдсан тул 10 мөр болсон. Хамгийн цөөн covering array олох нь NP-хүнд бодлого тул практикт greedy ойролцоо шийдэл ашигладаг ч бүх хос хамрагдсан хэвээр.
- **3-way:** 20 мөр, бүрэн олонлог 144-ийн ойролцоогоор 14%. Хамрах зэрэг 2-оос 3 болоход мөрийн тоо 2 дахин өссөн ч бүх хослолыг шалгахаас хамаагүй цөөн.

### B2: Шилжүүлгийн model-ууд
- **Файлууд:** `transfer-nc.model` (хязгаарлалтгүй), `transfer.model` (A2-ын хязгаарлалттай), `transfer-neg.model` (`~` сөрөг утгатай).

`transfer.model`-ийн хязгаарлалт нь A2-ын давуу эрэмбэнээс (`from → to → amount`) гарсан:

```text
IF [From] <> "Valid" THEN [To] = "Valid" AND [Amount] = "Normal";
IF [To] <> "Valid" THEN [Amount] = "Normal";
