# Chapter 3
###  What is SQL ?

- Structured Query language .
- declarative not procedural .
- Main purpose: retrieve data from databases, but also used to create, modify, and delete database structures and records.
- standard for interact with relational database .
### SQL in web :
- in web we don't use sql .
- we use stored procedures instead :
	- more security .
	- reduce network usage .
## Syntax Components :
- Statements "Commands" in sql (used to tell RDBMS what we want) split to 3 types :
	- *DDL (Data Definition Language) :* 
		- used to define structure (creation) of tables .
		- contains five main statements (CREATE table, DROP table, ALTER table, CREATE index, DROP index).
	- *DML (Data Manipulation Language) :*
		- used to manipulate & view data .
		- contains 4 statements (UPDATE, DELETE, INSERT, SELECT) .
	- *DCL (Data Control Language) :*
		- used to manage access, permissions, ownership of database objects .
### Create table :

#### 🔹 What does `CREATE TABLE` do?
- Used to define the structure of a **new table** inside an existing database.
- Syntax is **portable** across RDBMSs (unlike creating the database itself).
- Can be used من خلال:
    - **SQL command line**
    - **Scripts**
---
#### 🔹 Basic Syntax

```sql
CREATE TABLE <table-name> (
  <column-name> <data-type> [DEFAULT <value>] [<column-constraints>],
  ...
  [<table-constraints>]
);
```
#### ✅ Syntax Breakdown:

|Element|Description|
|---|---|
|`<table-name>`|اسم الجدول|
|`<column-name>`|اسم العمود|
|`<data-type>`|نوع البيانات (زي CHAR, INT...)|
|`DEFAULT <value>`|(اختياري) قيمة افتراضية للعمود|
|`<column-constraints>`|(اختياري) قيود العمود (زي NOT NULL, CHECK...)|
|`<table-constraints>`|(اختياري) قيود على الجدول بالكامل (زي PRIMARY KEY...)|
|`;`|نهاية الأمر (مهم جدًا في SQL)|

> ❗ **SQL is not case-sensitive**, but capitalizing keywords improves readability.
---
#### 🧪 Example – CUSTOMER Table

|CustomerNo|First|Last|Address|CreditLimit|
|---|---|---|---|---|
|CHAR(9)|CHAR(20)|CHAR(20)|VARCHAR(100)|SMALLINT|

#### 💻 SQL Code:

```sql
CREATE TABLE CUSTOMER (
  CustomerNo   CHAR(9)      NOT NULL,
  First        CHAR(20)     NOT NULL,
  Last         CHAR(20)     NOT NULL,
  Address      VARCHAR(100),
  CreditLimit  SMALLINT     CHECK (CreditLimit > 0),
  PRIMARY KEY (CustomerNo)
);
```

---
#### 🔍 Explanation of Constraints Used:

|Constraint|Purpose|
|---|---|
|`NOT NULL`|يمنع أن يكون العمود فارغًا|
|`CHECK (...)`|يحدد شرطًا منطقيًا يجب أن يتحقق لقيمة العمود|
|`PRIMARY KEY (...)`|يحدد العمود الذي يمثل المفتاح الأساسي (قيم فريدة + غير فارغة)|

- `CREATE TABLE` defines **table schema**.
- Use **data types** carefully.
- Constraints like `NOT NULL`, `CHECK`, `PRIMARY KEY` ensure **data integrity**.
- Separate items with **commas**, and finish with a **semicolon**.

---
### SQL-92 Data Types
#### 🔹 1. Numeric Data Types

|Type|Use for|Max Value (تقريبًا)|
|---|---|---|
|`SMALLINT`|أرقام صغيرة|حتى 32,767|
|`INTEGER`|أرقام صحيحة أكبر|حتى 2.1 مليار|
|`NUMERIC(p,s)`|أرقام عشرية بدقة ثابتة|`p`: digits total, `s`: digits after decimal|
|`FLOAT`|أرقام عشرية بفاصلة عائمة|غير دقيق 100%، مناسب للقياسات العلمية|

> ⚠ بعض المنتجات (زي Oracle) بتستخدم أنواعها الخاصة زي `NUMBER` بدل `NUMERIC`.

---
#### 🔹 2. Date and Time

|Type|Use for|
|---|---|
|`DATE`|التاريخ فقط (يوم/شهر/سنة)|

---
#### 🔹 3. Character Data Types

|Type|وصف|خصائص|
|---|---|---|
|`CHAR(n)`|نص بطول ثابت (Fixed-length)|يُملأ بمسافات إضافية إذا كانت القيمة أقصر من `n`|
|`VARCHAR(n)`|نص بطول متغير (Variable-length)|لا يُهدر مساحة، لكن أبطأ في المعالجة|

---
#### 🔍 Example from CUSTOMER Table:

`First  CHAR(20) Last   CHAR(20) Address VARCHAR(100)`
##### ✅ Interpretation:
- First/Last: دائمًا 20 حرف → أسرع للبحث.
- Address: طول متغير حسب الحاجة → أفضل لتوفير المساحة.
---
#### 🔄 CHAR vs VARCHAR – أيهما تختار؟

|الحالة|النوع الأنسب|
|---|---|
|القيم غالبًا بنفس الطول|`CHAR(n)`|
|الأعمدة يتم البحث فيها كثيرًا|`CHAR(n)`|
|نصوص طويلة / نادرة الاستخدام|`VARCHAR(n)`|
|توفير مساحة على حساب السرعة|`VARCHAR(n)`|

> ✳ مثال: أرقام الهواتف أو الأكواد تستخدم `CHAR`، بينما العناوين تستخدم `VARCHAR`

### Column & Table Constraint :

- **Column Constraints**: تنطبق على عمود واحد.
- **Table Constraints**: تنطبق على جدول كامل (وممكن تشتغل على أكثر من عمود مع بعض).
#### 🔹 ما هي القيود (Constraints)؟

> **قيود = قواعد تحكم البيانات**  
> تمنع إدخال بيانات خاطئة أو غير متوافقة مع منطق النظام.

|النوع|يطبق على|مثال|
|---|---|---|
|Column Constraint|عمود واحد فقط|`CreditLimit SMALLINT NOT NULL`|
|Table Constraint|جدول كامل (عدة أعمدة)|`PRIMARY KEY (A, B)`|

---
#### 🔑 أنواع القيود الأساسية:

|Constraint|الوظيفة|مثال|
|---|---|---|
|`PRIMARY KEY`|يحدد المفتاح الأساسي. يمنع التكرار والـ null|`PRIMARY KEY (CustomerNo)`|
|`FOREIGN KEY`|يحافظ على التكامل المرجعي (ربط بين جداول)|`FOREIGN KEY (OrderID) REFERENCES Orders(ID)`|
|`NOT NULL`|يمنع القيم الفارغة (null)|`FirstName CHAR(20) NOT NULL`|
|`UNIQUE`|يمنع التكرار في العمود (يستخدم كمفتاح بديل)|`Email VARCHAR(50) UNIQUE`|
|`CHECK (...)`|يفرض شرط منطقي على القيم|`CHECK (CreditLimit >= 0)`|
|`CONSTRAINT <name>`|تسمية القيد لسهولة التعامل معه لاحقًا|`CONSTRAINT CreditCheck CHECK (CreditLimit > 0)`|

---
#### 🔍 مثال عملي – جدول `CUSTOMER`:

```sql
CREATE TABLE CUSTOMER (
  CustomerNo   CHAR(9) PRIMARY KEY,
  First        CHAR(20) NOT NULL,
  Last         CHAR(20) NOT NULL,
  Address      VARCHAR(100) NOT NULL,
  CreditLimit  SMALLINT CHECK (CreditLimit >= 0)
);
```

##### ✅ شرح المثال:

- `CustomerNo`: مفتاح أساسي ⇒ تلقائيًا `NOT NULL + UNIQUE`.
- `First`, `Last`, `Address`: لا يُسمح بـ null. 
- `CreditLimit`: يُسمح بـ null، لكن لو دخلت قيمة ⇒ لازم ≥ 0.

---
#### ⚠ ملاحظات مهمة:

- **القيود المركبة** (Composite Key):  
    تستخدم أكثر من عمود كمفتاح، مثل:
    ```sql
    PRIMARY KEY (A, B, C)
    ```

- **null ≠ 0 أو ""**
    
    - `CHECK (x > 0)` لا تمنع `NULL`
    - الحل: استخدم `NOT NULL` بجانب `CHECK` إذا لزم الأمر.

- **عدم تكرار NOT NULL مع PRIMARY KEY**:
    - `PRIMARY KEY` بالفعل يمنع null ⇒ تكرار `NOT NULL` غير ضروري ويبطّئ الأداء.
        
- **CONSTRAINT** :
    - تسمية القيود مهم لو عايز تعدّلها أو تحذفها باستخدام `ALTER TABLE`.

---
#### ✅ أفضل الممارسات:

|الحالة|النصيحة|
|---|---|
|عمود ممكن يدخل في عمليات منطقية (AND/OR)|لا تسمح بـ null|
|تريد تقييد القيمة ضمن نطاق معين|استخدم `CHECK (value BETWEEN x AND y)`|
|عمود لازم يكون فريد ومفتاح بديل|استخدم `UNIQUE`|

---
### DROP table :
- used to delete table data & structure from data base .
- DROP table customer
- This **completely removes the table** — data, columns, constraints — everything .
---
### ALTER table :

يُستخدم لتعديل هيكل الجدول الموجود دون الحاجة لحذفه وإعادة إنشائه.

---

#### 📌 أنواع استخدامات `ALTER TABLE`:

##### 1. ➕ **ADD Form**

لإضافة عمود جديد أو قيد (Constraint) جديد.

###### 📄 صيغة:

```sql
ALTER TABLE table_name ADD COLUMN column_name data_type [DEFAULT value] [NOT NULL];
ALTER TABLE table_name ADD CONSTRAINT constraint_name ...;
```

###### 🧠 ملاحظات:

- الأعمدة الجديدة تُملأ بـ `NULL` أو بالقيمة الافتراضية (إذا تم تحديدها).
- لا يمكنك إضافة عمود `NOT NULL` إلا إذا أعطيته قيمة `DEFAULT`.
- لا يمكن في SQL-92 إضافة قيد `NOT NULL` على عمود موجود مباشرة (نستخدم طرق بديلة أو خصائص مخصصة في RDBMS حديثة).
###### 🧪 مثال:

```sql
ALTER TABLE EMPLOYEE ADD COLUMN ManagerID INTEGER DEFAULT 0 NOT NULL;
ALTER TABLE EMPLOYEE ADD FOREIGN KEY (ManagerID) REFERENCES EMPLOYEE(EmployeeID);
```

---

##### 2. 🛠️ **ALTER Form**

لتعديل الخصائص مثل القيمة الافتراضية.
###### 📄 صيغة:

```sql
ALTER TABLE table_name ALTER COLUMN column_name SET DEFAULT value;
ALTER TABLE table_name ALTER COLUMN column_name DROP DEFAULT;
```
###### 🧪 مثال:

```sql
ALTER TABLE EMPLOYEE ALTER COLUMN ManagerID SET DEFAULT 0;
```

---

##### 3. ❌ **DROP Form**
لحذف عمود أو قيد (Constraint) من الجدول.
###### 📄 صيغة:

```sql
ALTER TABLE table_name DROP COLUMN column_name;
ALTER TABLE table_name DROP CONSTRAINT constraint_name;
```
###### 🧠 ملاحظات:

- البيانات داخل العمود المحذوف تُفقد.
- لا يمكن حذف عمود مشارك في قيد (مثل Foreign Key) دون حذف القيد أولًا.
- يجب معرفة **اسم القيد** إذا أردت حذفه.

---

#### 🧩 مثال تطبيقي شامل:
##### 🧨 تعديل عمود `CreditLimit` من nullable إلى NOT NULL:
1. أضف عمود جديد `TempLimit`:
```sql
ALTER TABLE CUSTOMER ADD COLUMN TempLimit SMALLINT DEFAULT 0 NOT NULL CHECK (TempLimit >= 0);
```
2. انسخ البيانات إليه:
```sql
UPDATE CUSTOMER SET TempLimit = CreditLimit WHERE CreditLimit IS NOT NULL;
```
3. احذف العمود القديم:
```sql
ALTER TABLE CUSTOMER DROP COLUMN CreditLimit;
```
4. أنشئ العمود من جديد بنفس الاسم لكن بقيود جديدة:
```sql
ALTER TABLE CUSTOMER ADD COLUMN CreditLimit SMALLINT DEFAULT 0 NOT NULL CHECK (CreditLimit >= 0);
UPDATE CUSTOMER SET CreditLimit = TempLimit;
ALTER TABLE CUSTOMER DROP COLUMN TempLimit;
```
---
### CREATE & DROP INDEX

#### 🧠 ما هو الـ Index؟

الـ **Index** هو هيكل بيانات يُستخدم لتسريع عمليات البحث (SELECT) في الجداول، مثل الفهرس في كتاب لتسهيل الوصول للمحتوى.

---
##### 📌 أولًا: `CREATE INDEX`

###### 📄 الصيغة:

```sql
CREATE [UNIQUE] INDEX index_name ON table_name (column1, column2, ...);
```

##### 🔹 `UNIQUE` (اختياري):

- يستخدم إذا كنت تريد منع تكرار القيم في الأعمدة المفهرسة.    
- يُستخدم غالبًا مع الأعمدة التي تمثل **Primary Key** أو **قيود فريدة (Unique Constraints)**. 

##### 🧪 أمثلة:

```sql
-- فهرس عادي (غير فريد) على اسم العميل:
CREATE INDEX CustomerNameIndex ON CUSTOMER (Last, First);

-- فهرس فريد على رقم الموظف (بما أنه Unique):
CREATE UNIQUE INDEX EmployeeIDIndex ON EMPLOYEE (EmployeeID);
```

#### 🔎 ملاحظات:
- لا يُشترط أن تكون الأعمدة مفاتيح أساسية أو فريدة لتتم فهرستها.
- معظم قواعد البيانات **تُنشئ فهارس تلقائيًا على الـ Primary Keys** و الـ Unique constraints.

---

#### ❌ ثانيًا: `DROP INDEX`
##### 📄 الصيغة:
```sql
DROP INDEX index_name;
```
##### 🧪 مثال:
```sql
DROP INDEX CustomerNameIndex;
```
##### ⚠️ تحذير:
- حذف الفهرس **لا يحذف الأعمدة أو البيانات**، فقط يوقف تسريع الوصول المرتبط به.

---
#### 🧠 ملاحظات عامة:
- **الفهارس تُسرّع عمليات SELECT**، لكن قد تُبطئ عمليات `INSERT`, `UPDATE`, `DELETE` لأن النظام يحتاج إلى تحديث الفهرس أيضًا.
- **اختر الأعمدة المفهرسة بعناية** — لا تفهرس كل شيء.
---

### ✅ `INSERT` — إدخال بيانات جديدة

#### 🔹 الغرض:
إضافة صف جديد (Row) في جدول موجود مسبقًا.

---
#### 📄 الصيغة العامة:
```sql
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);
```
- يمكن حذف الجزء الخاص بالأعمدة، لكن **يفضَّل دائمًا كتابته لتجنب الأخطاء**.
- الترتيب مهم جدًا إذا حذفت أسماء الأعمدة.

---

#### 🧪 أمثلة:

##### ✅ مثال مكتمل:
```sql
INSERT INTO EMPLOYEE (EmployeeID, Name, Position, Location, Salary)
VALUES (617258, 'Jane Smith', 'Manager', 'London', 35000);
```

##### ✅ مثال بدون إدخال لقيمة المفتاح الأساسي (Auto-generated):
```sql
INSERT INTO EMPLOYEE (Name, Position, Location, Salary)
VALUES ('Heather Ralaton', 'Manager', 'Melbourne', 33000);
```
- في هذا المثال، قاعدة البيانات تولد `EmployeeID` تلقائيًا (مثلاً باستخدام `AUTO_INCREMENT` في MySQL أو `SERIAL` في PostgreSQL).
---

#### 🔍 استخدام القيم الافتراضية أو `NULL`

##### ✅ إدخال مع قيمة افتراضية أو `NULL`:
```sql
-- يتم إدخال NULL تلقائيًا إذا سمح العمود بها:
INSERT INTO CUSTOMER (CustomerNo, First, Last, Address)
VALUES ('5794-3467', 'Eric', 'Wilberforce', '9558 Great South Road');

-- يتم إدخال NULL يدويًا:
INSERT INTO SALE (SaleNo, SaleDate, CustomerNo, ProductNo, Qty, Amount, SalesRep)
VALUES (12348, '2002-08-13', NULL, 'DHU69863', 50, 118.5, 'Sara Thompson');
```

> 💡 إذا تم تقييد العمود بعدم قبول NULL، وكان له **قيمة افتراضية** (`DEFAULT`)، فسيتم استخدام هذه القيمة بدلًا من NULL.

---
#### 💬 ملاحظات مهمة:

|الملاحظة|الشرح|
|---|---|
|🔠 النصوص توضع بين علامات تنصيص `'`|مثل `'Jane Smith'`|
|❗ لا تخلط بين `'` و `"`|لأن بعضها يستخدم لأسماء الأعمدة (حسب الـ RDBMS)|
|⚠️ ترتيب القيم مهم جدًا|خصوصًا إذا لم تذكر أسماء الأعمدة|
|📅 التواريخ|يجب أن تكون بصيغة `YYYY-MM-DD` (وقد تختلف حسب قاعدة البيانات)|

---
ممتاز، دخلت الآن في ثاني عملية من عمليات CRUD وهي **`UPDATE`**، الخاصة بتعديل البيانات الموجودة بالفعل في قاعدة البيانات.

دعني ألخص لك هذا الدرس بشكل منظم ومبسط 👇

---

### ✅ `UPDATE` — تعديل البيانات

#### 🔹 الغرض:

تعديل بيانات موجودة في صفوف من جدول معين.

---

#### 📄 الصيغة العامة:

```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
[WHERE condition];
```

> 🟡 **`WHERE`** اختيارية، ولكن بدونها سيتم تعديل **كل الصفوف** في الجدول!

---

#### 🧪 أمثلة:

##### 🔸 تعديل كل الرواتب (جميع الصفوف):

```sql
UPDATE EMPLOYEE SET Salary = 37000;
```

> ⚠️ هذا يعدل **جميع** الموظفين!

---

##### 🔸 تعديل صف واحد باستخدام المفتاح الأساسي:

```sql
UPDATE EMPLOYEE SET Salary = 37000
WHERE EmployeeID = 617258;
```

> ✅ هذا آمن لأنه يستخدم مفتاح فريد.

---

##### 🔸 تعديل باستخدام الاسم (غير مفضل لأنه غير فريد):

```sql
UPDATE EMPLOYEE SET Salary = 37000
WHERE Name = 'Jane Smith';
```

> ❌ قد يغيّر أكثر من صف!

---

##### 🔸 تعديل أكثر من عمود:

```sql
UPDATE CUSTOMER 
SET CreditLimit = 0, Address = 'Iowa State Penitentiary'
WHERE CustomerNo = '4649-4673';
```

> ✅ تغيير أكثر من عمود في نفس العملية.

---

##### 🔸 استخدام NULL في التحديث:

```sql
UPDATE CUSTOMER 
SET CreditLimit = NULL, Address = 'Xowa State Penitentiary'
WHERE CustomerNo = '4649-4673';
```

> ✅ يمكنك استخدام `NULL` كقيمة جديدة، ولكن للمقارنة لا تستخدم `=` بل `IS NULL`.

---

##### 🔸 تعديل باستخدام عملية حسابية:

```sql
UPDATE PRODUCT 
SET Price = Price * 1.1;
```

> ✅ زيادة كل الأسعار بنسبة 10%. نستخدم نفس العمود كـ "مصدر" و"وجهة".

---

#### 💡 ملاحظات مهمة:

|الملاحظة|الشرح|
|---|---|
|⚠️ بدون WHERE = كارثة|يحدث تعديل لكل الصفوف|
|📛 استخدم مفتاح فريد في WHERE|مثل `ID` أو `CustomerNo`|
|🧮 يمكن استخدام تعبيرات رياضية|مثل `Price * 1.1`|
|❌ لا تستخدم `=` مع NULL للمقارنة|بل استخدم `IS NULL`|

---

ممتاز، الآن وصلت لآخر عملية في CRUD وهي **`DELETE`**، المسؤولة عن حذف البيانات من قاعدة البيانات.

دعني ألخص لك هذا الدرس بشكل واضح ومنظم:

---

### 🗑️ `DELETE` — حذف البيانات

#### 🔹 الغرض:
حذف صف (أو أكثر) من جدول موجود.

---
#### 📄 الصيغة العامة:
```sql
DELETE FROM table_name
[WHERE condition];
```

> 🟡 مثل `UPDATE`، إذا **لم تكتب `WHERE`** سيتم حذف **جميع الصفوف** في الجدول!

---
#### 🧪 أمثلة:
##### 🔸 حذف منتج باستخدام رقم المنتج:
```sql
DELETE FROM PRODUCT 
WHERE ProductNo = 'AAD62726';
```

> ✅ أفضل استخدام العمود الفريد مثل `ProductNo`.

---
##### 🔸 حذف موظفين براتب ≥ 40000 في نيويورك:
```sql
DELETE FROM EMPLOYEE 
WHERE Location = 'New York' AND Salary >= 40000;
```

> ✅ استخدام شرطين بربط `AND`.

---
##### 🔸 حذف مبيعات بدون رقم زبون:
```sql
DELETE FROM SALE 
WHERE CustomerNo IS NULL;
```

> ✅ عند التعامل مع `NULL`، نستخدم `IS NULL` بدلًا من `= NULL`.

---
##### 🔸 حذف كل المنتجات (بدون شرط):
```sql
DELETE FROM PRODUCT;
```

> ⚠️ **يحاول حذف كل الصفوف**، لكن:
- إذا كان هناك قيود **مفتاح خارجي (foreign key)** من جدول آخر (مثلاً `SALE` يعتمد على `PRODUCT`)، العملية قد **تفشل** للحفاظ على **سلامة البيانات** (referential integrity).

---
#### 🔒 ماذا تعني **Referential Integrity**؟
- يعني أن **الصفوف المرتبطة بمفاتيح خارجية** يجب أن تشير إلى صفوف صحيحة موجودة.
- إذا حاولت حذف صف من جدول أساسي (مثل `PRODUCT`) وكان هناك صف في جدول آخر (مثل `SALE`) يعتمد عليه، فإن قاعدة البيانات **ستمنع الحذف** تلقائيًا.

---
#### 💡 ملاحظات سريعة:

|الملاحظة|الشرح|
|---|---|
|⚠️ `DELETE` بدون `WHERE` = حذف شامل|لا تستخدمه إلا وأنت متأكد|
|✅ استخدم مفاتيح فريدة في الشرط|مثل `ProductNo`, `CustomerID`|
|❗ `IS NULL` بدلًا من `= NULL`|عند التعامل مع بيانات مفقودة|
|🔄 يتم التحقق من السلامة المرجعية|لمنع حذف بيانات يعتمد عليها جداول أخرى|

---

|المقارنة|`DELETE`|`TRUNCATE`|
|---|---|---|
|ينتمي لـ SQL رسميًا؟|✅ نعم|❌ لا (لكن مدعوم من أغلب قواعد البيانات)|
|الأداء|أبطأ — يسجل كل حذف|أسرع — لا يسجل كل حذف|
|يمكن استخدام `WHERE`؟|✅ نعم|❌ لا — يحذف كل الصفوف دائمًا|
|يسجل في Log؟|✅ نعم|❌ لا|
|يعيد تسلسل ID؟|❌ لا|✅ غالبًا نعم|

---
### 🔍 WHERE في SQL

#### 📌 ما هي؟
هي الجملة التي تُستخدم لتحديد **أي الصفوف (rows)** يجب أن تتأثر بالأمر (`UPDATE`, `DELETE`, `SELECT`).

---
#### 💥 لماذا WHERE مهمة جدًا؟
##### ✅ بدون `WHERE`:
```sql
DELETE FROM EMPLOYEE;
```
- هذا الأمر **يحذف جميع الموظفين** من الجدول! 

---
##### ✅ مع `WHERE`:
```sql
DELETE FROM EMPLOYEE WHERE Department = 'Sales';
```
- هذا يحذف فقط الموظفين في قسم المبيعات.

---
#### ⚠️ تحذير للمبتدئين
- نسيان جملة `WHERE` هو **أحد أكثر الأخطاء شيوعًا** في SQL!
- حتى إن كتبتها، فقد تكون **غير دقيقة** وتؤدي لتعديل أو حذف خاطئ.

---

#### ✅ كيف تختبر جملة WHERE بأمان؟

> لا تجرب جملة WHERE مباشرة مع `UPDATE` أو `DELETE` على قاعدة بيانات حقيقية (production).

##### 🧪 اختبرها أولًا هكذا:
```sql
SELECT * FROM EMPLOYEE WHERE Department = 'Sales';
```
- هذا يريك **ما هي الصفوف التي ستتأثر** دون حذف أو تعديل فعلي.

---
#### 🧠 كيف تعمل WHERE خلف الكواليس؟
- يتم تنفيذ التعبير الموجود بعد `WHERE` على كل صف (row) في الجدول.
- إذا أعطى **True** → الصف يُؤثَر عليه.
- إذا أعطى **False** → الصف يُتجاهل.

---

#### 🔗 يمكنك استخدام شروط منطقية مركبة:
```sql
UPDATE PRODUCT
SET Price = Price * 2
WHERE ((QtyInStock > 5000) OR (Price <= 10))
AND ReorderLevel < 1000;
```

##### هذا الشرط المعقد يعني:
- زد السعر للضعف إذا:
    - كان المخزون > 5000 **أو** السعر ≤ 10
    - **و** مستوى إعادة الطلب أقل من 1000

---

#### 🔥 مثال على DELETE مع شرط مركب:
```sql
DELETE FROM SALE
WHERE (
  (SaleDate BETWEEN '2002-08-13' AND '2002-08-14') AND (Salesrep = 'Li Qing')
  OR (ProductNo <> 'DHU69863')
  OR (Qty > 50)
);
```

##### سيتم حذف أي صف يُحقق:
- مبيعات قام بها Li Qing بين 13 و 14 أغسطس 2002.
- أو المنتج ليس هو DHU69863.
- أو كمية البيع > 50.

---

#### 🧪 رموز مفيدة في WHERE

|الرمز|المعنى|
|---|---|
|`=`|يساوي|
|`<>`|لا يساوي (قياسي)|
|`!=`|لا يساوي (بعض الأنظمة)|
|`BETWEEN`|بين قيمتين|
|`IS NULL`|القيمة فارغة|
|`AND`|كلا الشرطين يجب أن يتحققا|
|`OR`|يكفي أن يتحقق أحد الشرطين|
|`NOT`|نفي الشرط|

---
#### 🧠 ملخص ونصائح ذهبية:
- ✅ دائمًا اختبر `WHERE` باستخدام `SELECT`.
- ⚠️ لا تنسَ `WHERE` عند استخدام `DELETE` أو `UPDATE`.
- 🧪 استخدم قاعدة بيانات اختبارية عند تجربة شروط معقدة.
- 💾 خذ نسخ احتياطية قبل أي تعديل كبير.
---
### 🔹 Querying in SQL: SELECT

#### 🟦 شرح مبسط:

كل الاستعلامات (queries) في SQL تتم عن طريق الأمر `SELECT`.  
الهدف من الاستعلام هو إنك تطلب بيانات من قاعدة البيانات، ومين بيحدد شكل البيانات اللي راجعة؟ أنت، من خلال تحديد الأعمدة (columns)، الجداول (tables)، والشرط (conditions).

#### 🛠️ الصيغة الأساسية:
```sql
SELECT <columns>
FROM <tables>
[WHERE <conditions>];
```

#### ✳️ مثال:
لو عايز تطبع أسماء ورواتب الموظفين اللي شغالين في باريس:
```sql
SELECT Name, Salary
FROM EMPLOYEE
WHERE Location = 'Paris';
```

#### 📌 ملاحظات:
- **SELECT**: بتحدد الأعمدة اللي عايزها في النتيجة.
- **FROM**: بتحدد الجدول اللي هتستعلم منه.
- **WHERE**: شرط لتصفية النتائج (مش إجباري، ممكن تستغنى عنه).

---

#### 🔹 شرح SELECT * (الكل)
##### 🟦 شرح مبسط:
لما تستخدم `SELECT *`، معناها "هات كل الأعمدة من الجدول".  
مثال:
```sql
SELECT * FROM EMPLOYEE;
```

##### ✳️ نتيجة الاستعلام:
كل صفوف وأعمدة جدول EMPLOYEE هتظهر.
#### ⚠️ تحذير:
- استخدام `SELECT *` يعتبر **ممارسة سيئة (bad practice)** في عالم قواعد البيانات.
- بيستهلك **وقت وسعة سيرفر** لو الجدول كبير.
- ممكن يعرض **بيانات حساسة** لو تم إضافة أعمدة جديدة لاحقًا بدون قصد.

---

#### 🔹 فهم ترتيب التنفيذ في SELECT
##### 🟦 ترتيب التنفيذ المنطقي:
1. **FROM**: تحديد الجداول.
2. **WHERE**: تصفية الصفوف.
3. **SELECT**: تحديد الأعمدة اللي هتظهر.

> بالتالي، تقدر تستخدم أعمدة في WHERE حتى لو مش ظاهرة في SELECT.

---
#### 🔹 أمثلة تطبيقية على SELECT:
##### ✅ استعلام: الموظفين في لندن
```sql
SELECT Name FROM Employee
WHERE Location = 'London';
```
##### ✅ استعلام: رصيد منتج اسمه Grunge nut
```sql
SELECT ProductNo, Description, QtyinStock
FROM PRODUCT
WHERE ProductNo = 'FGE91822';
```
##### ✅ استعلام: العملاء اللي عندهم حد ائتماني أكبر من 6000
```sql
SELECT CustomerNo, First, Last
FROM CUSTOMER
WHERE CreditLimit > 6000;
```

---

#### 🔹 استعلام مبيعات خلال يومين
##### ✅ باستخدام OR:
```sql
SELECT SaleNo, SaleDate, Amount
FROM SALE
WHERE SaleDate = '2002-08-12' OR SaleDate = '2002-08-13';
```

##### ✅ باستخدام BETWEEN:
```sql
SELECT SaleNo, SaleDate, Amount
FROM SALE
WHERE SaleDate BETWEEN '2002-08-12' AND '2002-08-13';
```

##### ✅ باستخدام IN:
```sql
SELECT SaleNo, SaleDate, Amount
FROM SALE
WHERE SaleDate IN ('2002-08-12', '2002-08-13');
```

##### 💡 ملحوظة:
- لازم تكتب التواريخ بصيغة تناسب قاعدة البيانات بتاعتك (مثلاً `YYYY-MM-DD`).
- **IN** مفيد جدًا لو عندك لستة قيم.

---
#### 🔹 استعلام مبيعات عميل اسمه Amelia Waverley

##### ❌ الطريقة الغلط:
```sql
SELECT SaleNo, SaleDate, Amount, Salesrep
FROM CUSTOMER, SALE
WHERE First = 'Amelia' AND Last = 'Waverley';
```

ده بيطلع كل التركيبات الممكنة بين جدول CUSTOMER وSALE → يعني بيانات كتير جدًا ومغلوطة.
##### 🟦 ليه؟
لأن استخدام أكتر من جدول في `FROM` من غير `JOIN` بيعمل **Cartesian Product** (الضرب الديكارتي).

---

#### ✅ الطريقة الصحيحة: باستخدام JOIN

> نستخدم العلاقة بين الجداول (مفتاح أساسي ومفتاح أجنبي) للربط بينهم.

```sql
SELECT SaleNo, SaleDate, Amount, Salesrep
FROM CUSTOMER
JOIN SALE ON CUSTOMER.CustomerNo = SALE.CustomerNo
WHERE First = 'Amelia' AND Last = 'Waverley';
```

##### 🔍 التوضيح:
- `JOIN`: بيربط جدول CUSTOMER بجدول SALE على أساس CustomerNo.
- `WHERE`: بيحدد اسم العميل اللي احنا بندور عليه.

---

## JOINS :

### 📌 أولًا: لماذا نحتاج Joins؟

لأن قواعد البيانات مصممة بطريقة **التطبيع (Normalization)**، فالمعلومات موزعة على أكثر من جدول. لذلك إذا أردنا الحصول على بيانات متكاملة (مثلاً: اسم العميل وتفاصيل المشتريات)، نحتاج إلى **ضم الجداول (Join)** للحصول على نتيجة موحدة.

---

### 🔗 أنواع الـ Joins:

### 1. Condition-Based Joins (الانضمام بشرط)

وهو النوع الأكثر استخدامًا والأكثر وضوحًا.
#### 🧾 Pre-SQL-92 Style (الأسلوب القديم):
- نضع شروط الربط في جملة `WHERE`.
- مثال:
    ```sql
    SELECT SaleNo, SaleDate, Amount, Salesrep
    FROM CUSTOMER, SALE
    WHERE CUSTOMER.CustomerNo = SALE.CustomerNo
      AND First = 'Amelia' AND Last = 'Waverley';
    ```

#### ✅ SQL-92 Style (الأسلوب الحديث الموصى به):

- نستخدم الكلمة المفتاحية `JOIN`، وشرط الربط باستخدام `ON`.
- أكثر وضوحًا وسهولة في القراءة.
- مثال:
    ```sql
    SELECT SaleNo, SaleDate, Amount, Salesrep
    FROM CUSTOMER
    JOIN SALE ON CUSTOMER.CustomerNo = SALE.CustomerNo
    WHERE First = 'Amelia' AND Last = 'Waverley';
    ```

> ✨ الأفضل دائمًا استخدام SQL-92 لأنه أكثر قابلية للفهم والصيانة.

---
### 3. Outer Joins (الانضمام الخارجي)
تُستخدم عندما تريد عرض **جميع الصفوف من أحد الجداول**، حتى لو لم يكن لها تطابق في الجدول الآخر.
#### 🡸 Left Outer Join:
- يعرض **كل الصفوف من الجدول الأيسر** حتى لو لم يكن لها تطابق في الجدول الأيمن.
- مثال:
    ```sql
    SELECT PRODUCT.ProductNo, PRODUCT.Description, SALE.SaleNo, SALE.Amount
    FROM PRODUCT LEFT OUTER JOIN SALE
    ON PRODUCT.ProductNo = SALE.ProductNo;
    ```

#### 🡺 Right Outer Join:
- يعرض **كل الصفوف من الجدول الأيمن** حتى لو لم يكن لها تطابق في الجدول الأيسر.

#### ⬌ Full Outer Join:
- يعرض **كل الصفوف من كلا الجدولين**، سواء تطابقت أم لا (نادراً ما يُستخدم).

---

### ✅ نصائح عامة:
- استخدم **أسماء أعمدة واضحة** لتجنب اللبس.
- من الأفضل دائمًا **تحديد أسماء الجداول مع الأعمدة** (مثل `SALE.SaleNo`) حتى يكون الكود واضحًا.
- احرص على أن تكون العلاقات بين الجداول واضحة (عبر **مفاتيح أساسية وأجنبية**).
- تجنب استخدام `SELECT *` و `NATURAL JOIN` في الإنتاج، فهي قد تسبب نتائج غير متوقعة.

---
### 🎯 ملخص سريع:

|النوع|هل موصى به؟|ماذا يفعل؟|
|---|---|---|
|`NATURAL JOIN`|❌|يربط الجداول تلقائيًا حسب الأعمدة المشتركة (خطير)|
|`INNER JOIN`|✅|يعرض فقط الصفوف المتطابقة في الجدولين|
|`LEFT OUTER JOIN`|✅|يعرض كل الصفوف من الجدول الأيسر، والمتطابقة من الأيمن|
|`RIGHT OUTER JOIN`|✅|يعرض كل الصفوف من الجدول الأيمن، والمتطابقة من الأيسر|
|`FULL OUTER JOIN`|🔁|يعرض كل الصفوف من كلا الجدولين، مع Nulls في حال عدم وجود تطابق|

---
### 🔍 المشكلة الأساسية: Joins مقابل الأداء
-  ضرورية في قواعد البيانات **المطبقة لل (normalization) لتجنب التكرار.
- لكن في بيئة الويب، الأداء له أولوية قصوى لأن **سرعة استجابة الموقع** تؤثر مباشرة على تجربة المستخدم وربما أرباح الشركة.
- **الانضمامات المعقدة (Joins)** خاصةً مع **بيانات ضخمة** (مثل آلاف العملاء وملايين المبيعات) قد تبطئ التطبيق.

---
### 💡 الحل: Pre-compute joins

#### ✅ عندما يكون الأداء أهم من التطبيع الكامل:
- إذا كانت البيانات **ثابتة أو شبه ثابتة** (مثل سجل مبيعات قديمة أو درجات طلاب تم اعتمادها)، يمكن:
    - **تحضير نتائج الـ Join مسبقًا**.
    - **تخزين النتيجة في جدول جديد** يحتوي على بيانات مكررة لكنها جاهزة للعرض بسرعة.

---
#### 🧱 مثال عملي: جدول SALE_HISTORY

##### 1. يتم إنشاؤه مرة واحدة:
```sql
CREATE TABLE SALE_HISTORY (
  CustomerNo CHAR(9),
  First CHAR(20) NOT NULL,
  Last CHAR(20) NOT NULL,
  Address VARCHAR(100) NOT NULL,
  ProductNo CHAR(8),
  Description VARCHAR(50) NOT NULL,
  Price NUMERIC(5,2) NOT NULL CHECK (Price >= 0 and Price < 999.99),
  SaleNo SMALLINT,
  SaleDate DATE NOT NULL,
  Qty INTEGER NOT NULL CHECK (Qty > 0),
  Amount NUMERIC(6,2) NOT NULL CHECK (Amount >= 0),
  PRIMARY KEY (SaleNo)
);
```

##### 2. ثم تعبئته ببيانات مشتقة من Join بين الجداول الأصلية:
```sql
INSERT INTO SALE_HISTORY (SaleNo, SaleDate, Qty, Amount, ProductNo,
                          Description, Price, CustomerNo, First, Last, Address)
SELECT SALE.SaleNo, SALE.SaleDate, SALE.Qty, SALE.Amount,
       PRODUCT.ProductNo, PRODUCT.Description, PRODUCT.Price,
       CUSTOMER.CustomerNo, CUSTOMER.First, CUSTOMER.Last, CUSTOMER.Address
FROM SALE, PRODUCT, CUSTOMER
WHERE SALE.ProductNo = PRODUCT.ProductNo
  AND SALE.CustomerNo = CUSTOMER.CustomerNo;
```
##### 🔁 بعدها، يُحدَّث هذا الجدول دوريًا:
- باستخدام **Triggers** أو **Scheduled Jobs**.
- أو باستخدام مفهوم **Materialized View**.

---

### 🎓 أمثلة تطبيقية شائعة
- سجل مشتريات العميل في موقع تسوّق.
- كشف درجات طالب في جامعة.
- فواتير العميل في موقع خدمات.

---
### 📝 ملاحظات إضافية:
- بعض قواعد البيانات مثل SQL Server تدعم اختصارات مثل `SELECT INTO` لإنشاء جداول مباشرة من الاستعلام (لكن هذا غير قياسي).
- التكرار في البيانات هنا **مبرر** إذا كان الهدف تحسين سرعة استرجاعها دون الحاجة لـ Join في كل مرة.
- هذا لا يلغي أهمية التطبيع، لكنه يدعو إلى **حل وسط ذكي**.

---

## 📊 1. Reporting with SQL: Key Features

### 🔹 A. Sorting with `ORDER BY`
- **Purpose**: Arrange the output rows in a desired sequence.
- **Syntax**:
    ```sql
    SELECT <columns>
    FROM <table>
    ORDER BY <column1> [ASC|DESC], <column2> [ASC|DESC];
    ```
- **Examples**:
    - Sort by position (ascending):
        ```sql
        SELECT Name, Position FROM EMPLOYEE ORDER BY Position;
        ```
        
    - Sort by salary (descending):
        ```sql
        SELECT Name, Salary FROM EMPLOYEE ORDER BY Salary DESC;
        ```
        
    - Sort by position (descending), then salary (ascending):
        ```sql
        SELECT Name, Position, Salary 
        FROM EMPLOYEE 
        ORDER BY Position DESC, Salary;
        ```
### 🔹 B. Removing Duplicates with `DISTINCT`

- **Purpose**: Get unique values from a column (or combination of columns).
- **Syntax**:
    ```sql
    SELECT DISTINCT <column> FROM <table>;
    ```
    
- **Example**:
    - Get unique sale dates:
        ```sql
        SELECT DISTINCT SaleDate FROM SALE;
        ```

---

### 📚 2. Real-World Use in Web Applications

#### ✅ Why It Matters:

- **Efficient display**: Reduces redundant data shown to users (e.g., repeated dates, names, etc.).
- **Better UX**: Sorting improves readability of reports or dashboards.
- **Performance-aware**: Aggregation and sorting can be expensive on large datasets — pre-computing or caching results is often needed.

---
### 🛠️ Tips for Web Apps:

- **Use sorting and filtering on backend (SQL) when possible**, instead of fetching raw data and sorting in the front end.
- **Combine `DISTINCT` with `ORDER BY`** when you want a clean, sorted list of unique values.

---
## 📊 2. Aggregation in SQL

### 🧮 Built-in Aggregate Functions

Aggregate functions summarize data from multiple rows into a single result:

|Function|Description|
|---|---|
|`COUNT(*)`|Total number of rows|
|`SUM(column)`|Total sum of values in a column|
|`MIN(column)`|Smallest value in a column|
|`MAX(column)`|Largest value in a column|
|`AVG(column)`|Average value in a column|

---
### 🧾 Basic Example

> How many employees do we have? What's the total, minimum, maximum, and average salary?

```sql
SELECT
  COUNT(*) AS total_employees,
  SUM(Salary) AS total_salary,
  MIN(Salary) AS min_salary,
  MAX(Salary) AS max_salary,
  AVG(Salary) AS avg_salary
FROM EMPLOYEE;
```

**🧠 Notes:**
- All functions return a **single result**.
- These apply to the **entire table** unless restricted with `WHERE`.
- Null values are **ignored** by all except `COUNT(*)`.

---

### 🧮 Using Expressions Inside Aggregates

> What's the total value of our inventory?

```sql
SELECT SUM(QtyInStock * Price) AS total_inventory_value
FROM PRODUCT;
```
You can use arithmetic expressions inside aggregate functions!

---

### 🧍‍♂️ COUNT with DISTINCT

> How many unique job positions are there?

```sql
SELECT COUNT(DISTINCT Position) AS unique_positions
FROM EMPLOYEE;
```

**✅ Use Cases:**
- Avoid counting duplicate values.
- Helps generate cleaner reports.

---

### ⚠️ Important Rule

You **can't mix** aggregate functions and regular columns unless you use a `GROUP BY` clause.

❌ Invalid:
```sql
SELECT COUNT(*), Name FROM EMPLOYEE; -- ❌ Syntax Error
```

✅ Valid only with grouping:
```sql
SELECT Position, COUNT(*) FROM EMPLOYEE GROUP BY Position;
```

---

### 🔍 **What Does `COUNT(*)` Mean?**
- Counts **all rows**, even if some columns have `NULL`.
- Doesn't rely on any specific column’s values.
- Fastest way to get total row count.

Use `COUNT(DISTINCT column)` if:
- You want to **ignore duplicates**.
- You only want to **count non-null unique** values.

---

### ✅ Why Aggregation Matters in Web Apps

- Useful for dashboards: totals, averages, summaries.
- Reduces the need for app-side calculations.
- Helps with performance if done in SQL instead of code.

---
## 📂 3. Grouping Rows with `GROUP BY`

### 🧠 What is `GROUP BY`?

`GROUP BY` is used to **group rows** that have the **same value** in one or more columns, often to apply **aggregate functions** (`COUNT`, `SUM`, etc.) **per group**.

---

#### 🔢 Basic Example

> How many employees do we have **in each position**?

```sql
SELECT Position, COUNT(*) AS employee_count
FROM EMPLOYEE
GROUP BY Position;
```

|Position|employee_count|
|---|---|
|Analyst|1|
|Manager|1|
|Programmer|2|

**Explanation:**
- SQL groups by `Position`, then counts rows in each group.
- Every group produces **one row** in the result.

---

### 🧮 Grouping by Multiple Columns

> Group employees by **position and location**.

```sql
SELECT Position, Location
FROM EMPLOYEE
GROUP BY Position, Location;
```

This returns unique combinations like:
- Analyst + New York
- Manager + London
- Programmer + London
- Programmer + Paris

**Rule:** Any "regular" column in `SELECT` **must** be in `GROUP BY`.

---

### ❌ You Can’t Use Aggregate Functions in `WHERE`

> Find sales reps with total sales > 2000:

❌ Invalid:
```sql
SELECT Salesrep
FROM SALE
WHERE SUM(Amount) > 2000
GROUP BY Salesrep;
```
**Error:** You can’t use `SUM` or other aggregates in `WHERE`.

---

### ✅ Use `HAVING` Instead of `WHERE` for Group Conditions

✔️ Valid:

```sql
SELECT Salesrep
FROM SALE
GROUP BY Salesrep
HAVING SUM(Amount) > 2000;
```

**Explanation:**
- `GROUP BY` makes groups.
- `HAVING` filters **groups** based on aggregate conditions.

**🧠 Summary of Filtering:**

|Clause|Filters On|Works With Aggregates?|
|---|---|---|
|`WHERE`|Individual rows|❌ No|
|`HAVING`|Grouped rows|✅ Yes|

---

### ⚡ Performance Tip

If a `GROUP BY` is used **frequently** on large tables:
- You can **pre-compute** the result.
- Save it in a **separate summary table**.
- Useful if data is **static or slow-changing**.

---

### 💡 Real-Life Use Cases
- Total sales per product or rep.
- Users per country or plan type.
- Page views per day/week.
- Inventory value per category.

---
### ✅ Rules Recap
1. Columns in `SELECT` must be in `GROUP BY` (unless inside an aggregate).
2. Use `HAVING` to filter grouped results.
3. Don’t mix aggregates and regular columns without `GROUP BY`.

---

## ملاحظات مهمه :
### ✅ التعليقات في SQL
تقدر تكتب تعليقات داخل أو بجانب أي سطر كود باستخدام `--`.  
دي مفيدة جدًا لو بتحفظ الكود ده علشان تستخدمه لاحقًا، زي لما تكتبه داخل **stored procedures** (هنتكلم عنها في الفصل ٨).

---
### ✅ التعبيرات (Expressions) والثوابت (Constants) في SELECT
مش لازم تستخدم أسماء أعمدة أو دوال فقط في `SELECT`، تقدر تستخدم تعبيرات رياضية أو حتى ثوابت.
#### مثال على تعبير رياضي:

```sql
SELECT Description, QtyInStock - ReorderLevel FROM PRODUCT;
```

دي بتحسب الفرق بين الكمية الموجودة وكمية إعادة الطلب (يعني فاضل قد إيه قبل ما تحتاج تطلب تاني).
#### مثال على ثوابت:
```sql
SELECT Salesrep, 'has sold to', First, Last FROM SALE JOIN CUSTOMER ON ...;
```
كلمة `'has sold to'` هتظهر زي ما هي في كل صف، كأنها جملة ثابتة بتوضح العلاقة.

---
### ✅ إعادة تسمية الجداول (Table Aliasing)

ممكن تستخدم أسماء مختصرة للجداول علشان تسهّل الكتابة وتخلي الكود أوضح، خاصة لما تعمل `JOIN`.
#### مثال:
```sql
SELECT S.SaleNo, C.First FROM SALE AS S JOIN CUSTOMER AS C ON S.CustomerNo = C.CustomerNo;
```

> ملاحظة: `AS` اختيارية. ممكن تكتب `SALE S` بدل `SALE AS S`.

---
### ✅ الـ Self Join

أحيانًا عايز تربط الجدول بنفسه، زي مثلاً لما يكون عندك موظف ومديره في نفس جدول `EMPLOYEE`.
#### المشكلة:
```sql
SELECT Name, Name FROM EMPLOYEE JOIN EMPLOYEE ON ManagerID = EmployeeID;
```
ده هيطلعلك خطأ لإن النظام مش عارف أن الـ EMPLOYEE هنا معناها حاجتين مختلفتين.
#### الحل باستخدام alias:
```sql
SELECT E.Name, M.Name
FROM EMPLOYEE E JOIN EMPLOYEE M ON E.ManagerID = M.EmployeeID;
```
- `E` هنا بتمثل الموظف
- `M` بتمثل المدير

> ملاحظة مهمة: **الـ RDBMS ما بيعملش نسخة من الجدول لما تستخدم alias**. هي مجرد أسماء مؤقتة مش أكتر.

---
### ✅ إعادة تسمية الأعمدة (Column Aliasing)
مفيدة جدًا خصوصًا لما تستخدم دوال تجميعية أو تعبيرات، علشان تطلع بأسماء مفهومة.
#### مثال بدون alias:
```sql
SELECT AVG(Price), AVG(QtyInStock) FROM PRODUCT;
```
ده هيطلعلك عمودين من غير أسماء واضحة.
#### مثال باستخدام alias:
```sql
SELECT AVG(Price) AS AveragePrice, AVG(QtyInStock) AS AverageQty FROM PRODUCT;
```
> كمان تقدر تسمي ناتج أي تعبير رياضي:
```sql
SELECT Description, (QtyInStock - ReorderLevel) AS QtyRemaining FROM PRODUCT;
```

---
## 🧠 إزاي تكتب SELECT Query ؟

أنت مش بس محتاج تكتب الكويري، كمان لازم تتأكد إنه بيرجع **النتيجة الصح**.
عشان كده، في خطوات وتقنيات تقدر تتبعها:

---

### ✅ أولًا: **ابدأ بـ FROM**

> **"ابدأ دايمًا بـ الجداول اللي هتشتغل عليها."**
- اسأل نفسك: "أنا هحتاج أنهي جدول/جداول؟"
- لو أكتر من جدول → أكيد في **JOIN**.
    - اتأكد إنك كاتب **join condition** صح. 
    - واختار النوع المناسب من الـ join (INNER / LEFT / ...).
- لو هتعمل join لنفس الجدول → استخدم **alias (تسمية مؤقتة)** للجداول.

---
### ✅ ثانيًا: **حدد الصفوف المطلوبة → WHERE**

> "هسترجع أي صفوف؟"

- حدد شروط الفلترة اللي محتاجها → حطها في **WHERE**.
- مثلًا: `WHERE salary > 5000 AND department = 'IT'`
- اختبر الشروط دي كويس جدًا، لأن أي خطأ هنا هيجيبلك نتايج غلط.
    - جرّب كل شرط لوحده الأول.
    - بعدين ضيفهم واحد ورا التاني، واختبر كل مرة.

---
### ✅ ثالثًا: **هل في تجميع؟ → GROUP BY**

> "هل محتاج أجمع بيانات؟ زي عدد الطلبات لكل عميل؟"

- لو أه → استخدم **GROUP BY**.
- الأعمدة اللي بتجّمع بيها، لازم تكتبها في SELECT.
- لو محتاج تفلتر بعد التّجميع → استخدم **HAVING**.

مثال:
```sql
SELECT department, COUNT(*) 
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

---
### ✅ رابعًا: **ايه الأعمدة اللي هتظهر؟ → SELECT**

> "محتاج أعرض انهي بيانات؟"

- اختار الأعمدة اللي محتاجها في النتيجة.
- ممكن تستخدم دوال، expressions، aliases، إلخ.

---
### ✅ خامسًا: **عايز النتيجة تكون مترتبة؟ → ORDER BY**

> "هل محتاج أرتب النتائج؟"

- استخدم `ORDER BY` لترتيب النتيجة.
- ممكن ترتب تصاعدي (ASC) أو تنازلي (DESC).

---
### 💡 نصائح ذهبية لبناء Queries صح:

1. **استخدم بيانات اختبار كويسة.**
2. **قسّم الكويري لمراحل، واشتغل عليهم واحدة واحدة.**
    - ابدأ مثلًا بـ FROM + WHERE بس.
    - شوف النتيجة، لو تمام → كمّل GROUP BY.
3. **اختبر، اختبر، اختبر.**
    - جرب كويري بسيط.
    - بعدين زوّد شرط شرط.
    - كل خطوة لازم تتأكد إن النتيجة مظبوطة.

---
### 🧪 مثال عملي:
#### المطلوب:

"عايز أعرف عدد الموظفين في كل قسم، بشرط إن المرتب أكبر من 4000، وارتبهم حسب الاسم تنازليًا."
#### خطوات التحليل:

1. **FROM** → جدول الموظفين `employees`.
2. **WHERE** → `salary > 4000`.
3. **GROUP BY** → `department`.
4. **SELECT** → `department`, `COUNT(*)`.
5. **ORDER BY** → `department DESC`.
#### الكويري:
```sql
SELECT department, COUNT(*) AS NumEmployees
FROM employees
WHERE salary > 4000
GROUP BY department
ORDER BY department DESC;
```
