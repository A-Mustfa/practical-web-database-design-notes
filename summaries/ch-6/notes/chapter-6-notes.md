# Chapter 6: Implementing the Database

In this chapter, we take the **conceptual design** of the database and extend it by adding **physical characteristics**, ultimately producing the **data definition language (DDL)** used to create database tables.

---

## ✅ Objectives

To transform a conceptual design into a working physical database, we must consider:

1. Target Database System
2. Naming Conventions
3. Associative Entities for Many-to-Many Relationships
4. Data Types and Nullability
5. Natural vs. Surrogate Primary Keys
6. Foreign Keys and Relational Integrity
7. Table and Column Definitions
8. Database Indexes
9. Sample DDL
10. Sample SQL

---

## 1️⃣ Target Database System

When setting up a web application, decisions are made early about:

- Host (Internet provider)
- Operating system
- Scripting language
- **Target database system**

These decisions affect deployment and design steps like:

- Reserving server file space
- Configuring server connectivity (OS ↔ DB ↔ Web)
- Installing the web application
- Setting up security controls

> ⚠️ The choice of the **target database system must be finalized** before preparing DDL scripts.

---

## 2️⃣ Naming Conventions

While naming conventions may not significantly affect system functionality, they **enhance clarity**, **collaboration**, and **maintenance**.

###  Table Names

Some common conventions include:

- Prefixing tables with `tbl` (e.g., `tblCustomer`) — not recommended due to redundancy.
- Using **singular** (e.g., `Customer`) or **plural** (e.g., `Customers`) — **either is fine**, but be consistent.
  - Plural may sound more natural in queries: `SELECT * FROM Products`
- Avoid **reserved SQL keywords** like `Order`, `Date`, etc. Instead, use plural or alternative names.

✅ Recommended approach:
- Use meaningful, **singular** table names (e.g., `Product`, `Category`, `Customer`)
- **Avoid reserved keywords** by using modified names like `Orders` or `OrderDetails`

###  Column Names

- Start with attribute names used in the conceptual design.
- **Avoid reserved keywords** (`Date`, `Order`, etc.).

#### Example:

```sql
SELECT ID, Name
FROM Customer
WHERE City = 'Eureka';
```


* To avoid confusion when joining multiple tables,or adopt prefix-based naming to avoid repetition:

| Command | Description |
| --- | --- |
| Customer | `CustId` |
| Order | `OrdId` |
| Product | `ProdId` |


* you can use table-qualified column names:


```sql
SELECT Customer.ID, Customer.Name, Product.ID, Product.Name
FROM Customer
JOIN Orders ON Customer.ID = Orders.CustomerID
JOIN OrderItem ON Orders.ID = OrderItem.OrderID
JOIN Product ON OrderItem.ProductID = Product.ID
WHERE Customer.City = 'Eureka';

```

---

## 3️⃣ Associative Entities for Many-to-Many Relationships

When transforming the conceptual model to a physical one, it’s essential to identify **many-to-many relationships** and resolve them by introducing **associative entities**.

![Conceptual model before transformation.](/summaries/ch-6/imgs/ERD-1.jpg)

### 🧩 Example 1: Product–Category

- A product can belong to multiple categories.
- A category can contain multiple products.
- This is a **many-to-many** relationship.

✅ Create an associative entity named: **ProductCategory**

![Product–Category transformation.](/summaries/ch-6/imgs/ERD-2.jpg)

- **Product → ProductCategory**: one-to-many
- **Category → ProductCategory**: one-to-many

This enables a flexible many-to-many mapping between `Product` and `Category`.

### 🧩 Example 2: Product–Order
![Product–Order transformation.](/summaries/ch-6/imgs/ERD-3.jpg)

- A single order can include multiple products.
- A single product can be part of multiple orders.

✅ Create an associative entity named: **OrderItem**

![After transformation transformation.](/summaries/ch-6/imgs/ERD-4.jpg)

> Why "OrderItem" and not "ProductOrder"?  
> While both are technically correct, "OrderItem" is a **commonly used term** that emphasizes the items in an order. The entity still performs both roles: it tracks **products in an order** and **orders for a product**.

After adding these associative entities, the **updated physical model** includes `ProductCategory` and `OrderItem`.

---

## 4️⃣ Data Types and Nullability

Once the structure is defined, we must choose **appropriate data types** and determine **whether columns can be null**.

### 🔠 Character Data Types

| Data Type | Description | Size Defined By User | Storage(Bytes) | Fixed/Variable | Usage Example | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| `char(n)` | Fixed-length character string | `n` (1 to 255) | Always `n` bytes | Fixed | `char(10)` | Wastes space if actual data is shorter than `n`
| `varchar(n)` | Variable-length character string | `n` (1 to 65535)* | Varies by content + 1-2B | Variable | `varchar(255)` | Preferred for longer or descriptive fields; minimal overhead |

> [!NOTE]
> * Actual maximum n varies by DBMS and total row size
> * `varchar` is recommended over `char` in most cases due to space efficiency.
> *  `varchar` most commonly used for text fields (e.g., names, descriptions).
> * `char` more efficient in fixed length strings than `varchar` due to speed efficiency.


&nbsp;&nbsp;&nbsp;

### 🔢 Numeric Data Types
| Data Type | Description | Bytes | Value Range| Fractional Support | SQL Standard | Notes |
| --- | --- | --- | --- | :---: | :---: | --- |
| `tinyint` | Very small integer | 1 | -128 to 127 | ❌ | ❌ | Not standard; use with caution (low portability) | 
| `smallint` | Small integer | 2 | 	-32,768 to 32,767 | ❌ | ❌ | Good for IDs or small counts |
| `int` / `integer` | Regular integer | 4 | -2,147,483,648 to 2,147,483,647 | ❌ | ✅ | Commonly used for surrogate keys |
| `numeric(p,s)` | Fixed-point decimal number | Varies | 	Depends on `p` and `s` | ✅ | ✅ | Used for money or precision amounts (e.g., prices) |
| `float` | Approximate float number | 4-8 | Very wide range | ✅ | ✅ | Suitable for scientific values (not used in web store) |
| `real` | Approximate single-precision | 4 | Depends on DBMS | ✅ | ✅ | Similar to float; lower precision |

> [!TIP]
>  Use `numeric(p, s)` for monetary values like prices: Example: numeric(7, 2) supports up to 99999.99.

> [!WARNING]
> * Avoid `tinyint` for portability. Use int or smallint unless storage is very limited.
> * Do not use strings for monetary values.

&nbsp;&nbsp;&nbsp;

### 🕒 Date and Time Data Types
| Data Type | Description | Format Example | SQL Standard | Notes |
| --- | --- | :---: | :---: | :---: |
| `date` | Calendar date only | `2025-06-27` | ✅  | Stores year-month-day | 
| `time` | Time of day | `14:30:00` | ✅ | Stores hour-minute-second |
| `datetime` | Date + time (combined) | `2025-06-27 14:30` |❌ | SQL Server & MySQL-specific (non-standard) |

> [!NOTE]
> * datetime is not part of standard SQL, but widely supported by systems like SQL Server and MySQL.
> * Use date and time when the separation is needed for portability. 


&nbsp;&nbsp;

### 🚫 Nullability

**Null** = "no value" or "value unknown"

> ⚠️ Null is not the same as 0 or an empty string!

When designing columns:
- Use `NULL` when the value is **optional**.
- Use `NOT NULL` when the value is **mandatory**.

#### Good Use of NULL:
> A column like `ShippedDate` may be null if an order hasn’t shipped yet.

> ❌ Using fake dates (e.g., `31-12-2999`) is dangerous and led to real-world issues like the **Y2K bug**.

#### SQL Behavior:
- Aggregate functions **ignore nulls** by default.
- Queries may behave differently if null checks aren’t handled properly.

---
## 5️⃣ Natural vs. Surrogate Primary Keys

### 🔑 What is a Primary Key?

A primary key must:
1. Be **unique** for each row.
2. **Never be null**.

> 🔒 It ensures each row in a table is uniquely identifiable and fully populated.

&nbsp;

###  Natural Primary Key

A **natural key** uses one or more existing attributes from the real-world entity you're modeling.

- Examples: `Email`, `NationalID`, `ISBN`
- Derived from the conceptual design

✅ Best used when:
- The value is **inherently unique**
- It is **stable** over time
- It is **meaningful** to users or business logic

> ⚠️ Avoid if the attribute may change or is too long/complex.

&nbsp;

###  Surrogate Primary Key

A **surrogate key** is an artificial column added to a table to uniquely identify rows.

- Typically auto-generated (e.g., `ID`, `CustomerID`)
- Has **no intrinsic business meaning**
- Often implemented as `INTEGER`

✅ Best used when:
- No clear natural key exists
- The natural key is too long, non-unique, or volatile

Common auto-increment mechanisms:

| Database System   | Syntax/Feature     |
|-------------------|--------------------|
| MySQL             | `AUTO_INCREMENT`   |
| SQL Server        | `IDENTITY`         |
| PostgreSQL        | `SERIAL` / `BIGSERIAL` |

> ✅ Surrogate keys automatically satisfy both **uniqueness** and **not null** constraints.
>  
> 🚫 Rule of thumb: **Don’t expose surrogate keys** to users — they carry no business meaning.


---
### 6️⃣ Relational Integrity Rule

![After transformation transformation.](/summaries/ch-6/imgs/customer-order-1toM.jpg)

The database enforces that:
- **Every `ordcustID` must match a valid `custID`**
- **No order can exist without a known customer**

This means:
- A **customer must be inserted first** before adding their order
- An order row **cannot reference a customer** that does not exist

> ⚠️ Even if a form allows entering customer and order details at once, the backend must insert the customer first, then the order.

&nbsp;

### ❓ Can a Foreign Key Be Null?

Yes — in cases where the relationship is **optional**.

Example:
- A product may have an optional `SupplierID` — if no supplier exists yet, it can be `NULL`.

But in our case:
- `ordcustID` must **not be null**
- Every order must belong to a customer

&nbsp;

### ❌ Deletion & Integrity

Relational integrity **prevents deletion of referenced rows**.

If:
- A `Customer` row is referenced by any `Order` (via `ordcustID`),

Then:
- That `Customer` **cannot be deleted**

> 🛡️ This protects against "dangling references" where a foreign key points to a non-existent row.

---

## 7️⃣ Table and Column Definitions 

Below is the schema definition for the tables in the database. It includes data types, nullability, and brief descriptions of each column.

### 📋 Table: `Product`

| Column          | Data Type        | Nullability | PK  | FK  | Description                                                                 |
|------------------|------------------|-------------|-----|-----|------------------------------------------------------------------------------|
| `ProdID`         | `char(10)`       | **NOT NULL**| ✅  |     | Natural primary key; supplier-defined product code                          |
| `ProdName`       | `varchar(100)`   | **NOT NULL**|     |     | Name of the product                                                         |
| `ProdDescr`      | `varchar(500)`   | NULL        |     |     | Short description of the product                                            |
| `ProdLongDescr`  | `varchar(2000)`  | NULL        |     |     | Detailed long description (may require `TEXT` in some DBMS)                |
| `ProdSize`       | `varchar(10)`    | NULL        |     |     | Size label (e.g., "XL", "XXL")                                              |
| `ProdColour`     | `varchar(10)`    | NULL        |     |     | Color name or code (e.g., "Red", "Taupe")                                   |
| `ProdWeight`     | `smallint`       | NULL        |     |     | Product weight in pounds                                                    |
| `ProdPrice`      | `numeric(7,2)`   | **NOT NULL**|     |     | Unit price of the product                                                   |
| `ProdOnHand`     | `smallint`       | **NOT NULL**|     |     | Inventory count available                                                   |
| `ProdComments`   | `varchar(1000)`  | NULL        |     |     | Optional customer feedback or testimonials                                  |
| `ProdImage`      | `varchar(100)`   | NULL        |     |     | URL to product image (not the image itself)                                 |
| `ProdDiscount`   | `numeric(7,2)`   | **NOT NULL**|     |     | Discount amount applied to the product                                      |
| `ProdShipCost`   | `numeric(7,2)`   | **NOT NULL**|     |     | Shipping cost for the product                                               |


### 📝 Notes & Design Considerations

- **Primary Key**: `ProdID` is a natural key chosen for its relevance to real-world product codes. It must be unique and not null.
- **Text Fields**:
  - `ProdDescr` and `ProdLongDescr` are optional fields. The long description may exceed varchar limits in some databases; in MySQL, use `TEXT` instead of `varchar(2000)` if needed.
- **Storage & Efficiency**:
  - Use `varchar` instead of `char` for flexibility, even in small fields like `ProdSize` and `ProdColour`, as the space savings are minimal.
- **Weight**:
  - `ProdWeight` uses `smallint` since it's rare for products to exceed 32,767 lbs.
  - Adjust the unit (pounds vs. kg) depending on shipping needs.
- **Image Handling**:
  - Images are stored by URL (`ProdImage`), not as BLOBs, for better web app performance and simplicity.
- **Monetary Fields**:
  - Use `numeric(7,2)` for `ProdPrice`, `ProdDiscount`, and `ProdShipCost` to ensure decimal accuracy.


> 🔍 The Product table currently **does not have any foreign keys**, as it stands independently in this phase of design.

&nbsp;


### 📋 Table: `Category`

| Column         | Data Type      | Nullability   | PK  | FK  | Description                                                                 |
|----------------|----------------|---------------|-----|-----|-----------------------------------------------------------------------------|
| `CatID`        | `integer`      | **NOT NULL**  | ✅  |     | Surrogate primary key (auto-generated unique identifier for each category) |
| `CatName`      | `varchar(100)` | **NOT NULL**  |     |     | Name of the category                                                        |
| `CatDescr`     | `varchar(1000)`| NULL          |     |     | Optional long description of the category                                  |
| `CatParentID`  | `integer`      | NULL          |     | ✅  | Self-referencing foreign key to support category-subcategory hierarchy      |


### 📝 Notes & Design Considerations

- `CatID` is a **surrogate key**, typically auto-incremented by the database.
- `CatName` is required for every category and must be present.
- `CatDescr` is optional, allowing flexible additional descriptions.
- `CatParentID` allows recursive relationships:
  - If present, it must reference a valid `CatID`.
  - If `NULL`, the category is considered a **top-level category** (i.e., it has no parent).
  - This structure enables category nesting (e.g., "Electronics" → "Phones" → "Smartphones").

> 🔁  Recursive relationships like this are valid in relational models and help model hierarchical data.

&nbsp;

### 📋 Table: `ProductCategory`

| Column             | Data Type     | Nullability   | PK  | FK  | Refers To          | Description                                                                 |
|--------------------|---------------|---------------|-----|-----|---------------------|-----------------------------------------------------------------------------|
| `ProdCatProdID`    | `char(10)`    | **NOT NULL**  | ✅  | ✅  | `Product.ProdID`    | Product ID (foreign key and part of composite primary key)                  |
| `ProdCatCatID`     | `integer`     | **NOT NULL**  | ✅  | ✅  | `Category.CatID`    | Category ID (foreign key and part of composite primary key)                 |
| `ProdCatDefault`   | `char(1)`     | **NOT NULL**  |     |     |                     | Indicates default category for a product (`Y` / `N`)                        |


### 📝 Notes & Design Considerations

- This is an **associative (junction) table** to model the **many-to-many** relationship between products and categories.
- It has a **composite primary key**:
  - `ProdCatProdID` + `ProdCatCatID`
  - This ensures that a product can only belong to a specific category **once**.
- Both primary key components are also **foreign keys**, referencing:
  - `ProdCatProdID` → `Product.ProdID`
  - `ProdCatCatID` → `Category.CatID`
- `ProdCatDefault`:
  - Used to **mark the default category** for a product.
  - Chosen as `char(1)` with values like `Y` / `N` for **portability** (e.g., Boolean is not supported in all DBMS).

> ✅ Always ensure that the foreign key data types exactly match the referenced primary keys for compatibility and referential integrity.


&nbsp;


### 📋 Table: `Customer`

| Column         | Data Type       | Nullability   | PK  | FK  | Refers To | Description                                                               |
|----------------|-----------------|---------------|-----|-----|-----------|---------------------------------------------------------------------------|
| `CustID`       | `integer`       | **NOT NULL**  | ✅  |     |           | Surrogate primary key (auto-incremented identifier)                      |
| `CustUsername` | `varchar(20)`   | **NOT NULL**  |     |     |           | Unique username used for login                                           |
| `CustPassword` | `varchar(20)`   | **NOT NULL**  |     |     |           | Password (should be hashed in real applications)                         |
| `CustEmail`    | `varchar(100)`  | **NOT NULL**  |     |     |           | Email address (used for communication and recovery)                      |
| `CustFName`    | `varchar(50)`   | NULL          |     |     |           | First name (optional; some users may only have one legal name)          |
| `CustLName`    | `varchar(50)`   | NULL          |     |     |           | Last name (optional for flexibility)                                     |
| `CustAddress1` | `varchar(100)`  | NULL          |     |     |           | Primary street address                                                   |
| `CustAddress2` | `varchar(100)`  | NULL          |     |     |           | Secondary address line (e.g., apartment or suite number)                 |
| `CustCity`     | `varchar(100)`  | NULL          |     |     |           | City                                                                      |
| `CustProv`     | `varchar(100)`  | NULL          |     |     |           | Province, state, or region                                               |
| `CustCountry`  | `varchar(50)`   | NULL          |     |     |           | Country name                                                              |
| `CustPostal`   | `varchar(10)`   | NULL          |     |     |           | Postal or ZIP code                                                       |


### 📝 Notes & Design Considerations

- 🔐 **Authentication**:
  - `CustUsername` and `CustPassword` are mandatory for login purposes.
  - Passwords should be **securely hashed** in practice (not stored as plain text).
- 📧 **Contact**:
  - `CustEmail` is required for communications and account recovery.
- 👤 **Name Fields**:
  - First and last names are optional to support users with single or nontraditional names.
- 🏠 **Address Fields**:
  - All address fields are optional to support guest registrations or newsletter subscribers.
  - Using `varchar` fields avoids localization issues (e.g., country codes or region formats).
  - No foreign keys are enforced for country or region to keep flexibility for international addresses.

> 🌐 This flexible design allows customers to create accounts without placing orders, which supports marketing activities like newsletters or pre-registration.

&nbsp;

### 📋 Table: `Orders`

| Column         | Data Type       | Nullability   | PK  | FK  | Refers To        | Description                                                                   |
|----------------|------------------|---------------|-----|-----|------------------|-------------------------------------------------------------------------------|
| `OrdID`        | `integer`        | **NOT NULL**  | ✅  |     |                  | Surrogate primary key; uniquely identifies each order                        |
| `OrdTotal`     | `numeric(7,2)`   | **NOT NULL**  |     |     |                  | Total price of the order                                                     |
| `OrdDiscount`  | `numeric(7,2)`   | **NOT NULL**  |     |     |                  | Discount applied to the order                                                |
| `OrdTax`       | `numeric(7,2)`   | **NOT NULL**  |     |     |                  | Tax calculated for the order                                                 |
| `OrdShipping`  | `numeric(7,2)`   | **NOT NULL**  |     |     |                  | Shipping cost of the order                                                   |
| `OrdDate`      | `date`           | **NOT NULL**  |     |     |                  | Date when the order was placed                                               |
| `OrdShipDate`  | `date`           | NULL          |     |     |                  | Date the order was shipped (nullable if not yet shipped)                     |
| `OrdCustID`    | `integer`        | **NOT NULL**  |     | ✅  | `Customer.CustID`| Foreign key linking the order to the customer who placed it                 |


### 📝 Notes & Design Considerations

- `OrdID` is a **surrogate primary key**, typically auto-incremented by the database system.
- All **monetary columns** (`OrdTotal`, `OrdDiscount`, `OrdTax`, `OrdShipping`) are required:
  - Even if their value is `0.00`, they must be recorded.
- `OrdDate` is mandatory and usually set using the system's current date/time function at insert.
- `OrdShipDate` is **nullable**:
  - If the order hasn't been shipped yet, it remains `NULL`.
- `OrdCustID` is a foreign key pointing to the **customer who placed the order**.
  - Relational integrity ensures the customer exists before placing an order.

> 📦 This table is central to the system and supports full tracking of placed and pending orders with financial breakdown.

&nbsp;

### 📋 Table: `OrderItem`

| Column             | Data Type       | Nullability   | PK  | FK  | Refers To        | Description                                                                 |
|--------------------|------------------|---------------|-----|-----|------------------|-----------------------------------------------------------------------------|
| `OrdItemOrdID`     | `integer`        | **NOT NULL**  | ✅  | ✅  | `Orders.OrdID`   | Order ID this item belongs to (part of composite PK)                       |
| `OrdItemProdID`    | `char(10)`       | **NOT NULL**  | ✅  | ✅  | `Product.ProdID` | Product ID in this order item (part of composite PK)                       |
| `OrdItemQty`       | `smallint`       | **NOT NULL**  |     |     |                  | Quantity of the product ordered                                            |
| `OrdItemPrice`     | `numeric(7,2)`   | **NOT NULL**  |     |     |                  | Price per unit (may differ from standard product price)                    |
| `OrdItemDiscount`  | `numeric(7,2)`   | **NOT NULL**  |     |     |                  | Discount applied on this item                                              |
| `OrdItemTax`       | `numeric(7,2)`   | **NOT NULL**  |     |     |                  | Tax applied to this item                                                   |
| `OrdItemShipping`  | `numeric(7,2)`   | **NOT NULL**  |     |     |                  | Shipping cost for this item                                                |


### 📝 Notes & Design Considerations

- This is an **associative table** that connects `Orders` and `Product` in a **many-to-many** relationship.
- The **composite primary key** (`OrdItemOrdID`, `OrdItemProdID`) ensures each product is only listed once per order.
- All monetary columns allow for **line-level overrides** (e.g., discounts or special pricing).
- `OrdItemQty` is typically `smallint`, but could use a different numeric type if fractional quantities are allowed (e.g., meters of fabric).

> 🧾 Enables detailed tracking of what was purchased, in what quantity, at what price, and with what adjustments.

&nbsp;

### 📋 Overall Database scheme :
![Overall Database scheme.](/summaries/ch-6/imgs/overall-database.jpg)

---

## 8️⃣ Database Indexes

Indexes improve the efficiency of **read operations** in a relational database. They work like a sorted lookup table, allowing the database engine to **quickly locate rows** based on specific column values.

---

### 🔑 When to Use Indexes

There are two common categories of columns that benefit from indexes:

- ✅ **Primary Keys** — automatically indexed by the DBMS.
- ✅ **Foreign Keys** — should be indexed **if they are used in joins or frequent lookups**.
- ✅ **Other Columns** — index if used in `WHERE`, `JOIN`, or `ORDER BY`.

---

### 🧠 How SQL Queries Are Executed

While SQL is written like:

```sql
SELECT ...
FROM Customer
JOIN Orders ON ...
WHERE CustCity = 'Eureka'
```
The actual logical execution order is:
 1. `FROM` and `JOIN` – Tables are joined first.
 2. `WHERE` – Filters are applied **after** joining.
 3. `SELECT` – Columns are retrieved.
 4. `ORDER BY` – Results are sorted (if needed).

> ✅ However, optimizers may apply filters early (e.g., filter CustCity) if an index exists—this improves performance significantly.

&nbsp;
### 🔍 Sample Query and Index Use
```sql
SELECT CustID, CustName, ProdID, ProdName
FROM Customer
JOIN Orders ON CustID = OrdCustID
JOIN OrderItem ON OrdID = OrdItemOrdID
JOIN Product ON OrdItemProdID = ProdID
WHERE CustCity = 'Eureka';
```

To optimize this query:

| Table        | Column                  | Why Index It?                 |
|--------------|-------------------------|-------------------------------|
| Customer     | `CustCity`              |Frequently filtered in `WHERE` |
| Orders       | `OrdCustID`             |Used in `JOIN` with Customer   |
| OrderItem    | `OrdItemOrdID`          | Part of PK – already indexed  |
| Product      | `ProdID`                | Primary Key – already indexed |

&nbsp;

### 🧩 Composite Primary Keys and Indexes
A composite primary key (e.g., `PRIMARY KEY (OrderID, ProductID)`) acts like a two-level sort:

* First sorted by `OrderID`
* Then within each group, sorted by ProductID

✅ Efficient:

```sql
WHERE OrderID = 123
```

✅ Efficient:

```sql
WHERE OrderID = 123 AND ProductID = 'X01'
```

 ❌ Not efficient:

```sql
WHERE ProductID = 'X01'
```

> The database cannot use the composite index to search only by `ProductID` unless `OrderID` is also specified.

&nbsp;

### 🛠 Indexing Strategy for Composite Keys
If the second column in a composite PK is used frequently on its own, create a separate index:

```sql
CREATE INDEX idx_ProductID ON OrderItem(ProductID);
```
This improves performance when `ProductID` is queried without `OrderID`.

### ✅ Indexing Best Practices

> [!TIP]
> * Always index primary keys                           → Automatically done by DBMS
> * Index foreign keys                                  → Especially for frequent joins
> * Add indexes for heavily filtered columns            → Like `CustCity` in customer lookup
> * Add separate index for 2nd column of composite PKs	→ Prevent full scans
> * Use numeric columns in indexes when possible	      → Faster and smaller than strings
> * Don’t over-index	                                  → Indexes slow down inserts/updates


> [!NOTE]
> - Indexes boost read speed but may slow down write operations due to overhead.
> - Use indexes for columns with high read frequency, large tables, or complex joins.
> - Avoid indexing small lookup tables unless absolutely necessary.
> - Indexes are often built as B-trees and managed automatically by the DBMS.

---

## 9️⃣ SQL DDL Example: Table Creation and Indexing

This section illustrates how to use **SQL Data Definition Language (DDL)** to declare tables and indexes.

> 📌 DDL is not a separate language. It refers to the subset of SQL used for defining and modifying database schema objects such as `CREATE`, `ALTER`, and `DROP`.



## 📦 Example: Create `OrderItem` Table with Keys

```sql
CREATE TABLE OrderItem (
    OrdItemOrdID       INTEGER       NOT NULL,
    OrdItemProdID      VARCHAR(10)   NOT NULL,
    OrdItemQty         SMALLINT      NOT NULL,
    OrdItemPrice       NUMERIC(7,2)  NOT NULL,
    OrdItemDiscount    NUMERIC(7,2)  NOT NULL,
    OrdItemTax         NUMERIC(7,2)  NOT NULL,
    OrdItemShipping    NUMERIC(7,2)  NOT NULL,

    -- Composite Primary Key
    PRIMARY KEY (OrdItemOrdID, OrdItemProdID),

    -- Foreign Key Constraints
    FOREIGN KEY (OrdItemOrdID) REFERENCES Orders,
    FOREIGN KEY (OrdItemProdID) REFERENCES Product
);
```
### ✅ Explanation:

- The primary key is composite: (OrdItemOrdID, OrdItemProdID), ensuring that each product in an order is unique.
- The foreign keys link:
  - OrdItemOrdID → Orders  
  - OrdItemProdID → Product
- Standard SQL assumes foreign keys reference the primary key of the target table unless otherwise specified.

&nbsp;

### 🔍 Add Index on Second Column of Composite Key
```sql
CREATE INDEX OrdItemProdIndex
ON OrderItem (OrdItemProdID);
```

### ✅ Why add this index?

- The composite primary key creates an index on (OrdItemOrdID, OrdItemProdID).
- But if we run a query using only OrdItemProdID in the WHERE clause, the default index won't help.
- Creating a separate index on OrdItemProdID improves query performance when used without the order ID.

&nbsp;

❕ Indexing Considerations

> [!WARNING]
> * Duplicate Indexes	                           → Defining another index on a column already covered by a PK may hurt write performance (inserts/updates).
> * Auto Indexing                                  → Most DBMSs automatically create a unique index on the primary key.
> * Surplus Indexes                                → Adding redundant indexes will not speed up reads but will slow down writes due to double maintenance
> * Composite Key Indexing Limitation		→ Composite indexes only help queries that start with the first column of the key.
> * Always avoid defining another index on the first column of a primary key. Let the database handle it.

> [!NOTE]
> - Use `REFERENCES TableName(ColumnName)` syntax only when referencing a non-primary key column.
> - Most SQL dialects (PostgreSQL, MySQL, SQL Server, etc.) support similar syntax with minor variations (like `AUTO_INCREMENT` or `SERIAL`).


---

## 🔟 Sample SQL : Search for Products

### Overview
This section explains how to implement a search functionality for an e-commerce website, enabling users to search for products based on a term they enter, such as "T-shirt". The system retrieves relevant product details based on the search input.

###  Example SQL Query

```sql
SELECT ProdID, ProdName, ProdDescr, ProdLongDescr, ProdSize, 
       ProdColour, ProdWeight, ProdPrice, ProdOnHand, ProdComments, 
       ProdImage, ProdDiscount, ProdShipCost
FROM Product
WHERE ProdName LIKE '%T-shirt%'
   OR ProdDescr LIKE '%T-shirt%'
   OR ProdLongDescr LIKE '%T-shirt%'
```
&nbsp;

## Avoid Using `SELECT *`

```sql
SELECT *
FROM Product
WHERE ProdName LIKE '%T-shirt%'
   OR ProdDescr LIKE '%T-shirt%'
   OR ProdLongDescr LIKE '%T-shirt%'
```

> [!TIP]
> Instead of using `SELECT *` to fetch all columns from the database, explicitly list the columns you need. This approach:
>   - Improves readability and maintainability of the code.
>   - Prevents unnecessary data from being returned (e.g., long product descriptions that won’t be displayed in the search results).
>   - Enhances performance by fetching only the required data.

### ❓ Why List Columns Explicitly?
- **Self-Documenting:** Listing the columns makes it clear what data is being selected, improving code clarity.
- **Efficiency:** Avoids fetching large and irrelevant data fields (like detailed product descriptions) when they are not needed for the search results.

&nbsp;

## ✅ Use of `LIKE` in SQL Queries
The `LIKE` operator is used in SQL to perform pattern matching. The query will search for the specified term (e.g., "T-shirt") in multiple columns, using wildcards (e.g., `%`) to match the term anywhere within the column's text.
&nbsp;

### 🛠 Integration with Scripting Languages
The search input provided by the user (e.g., from a web form) is passed to the SQL query through a server-side scripting language (e.g., Perl, PHP, ASP, JSP). The scripting language handles:
- Capturing the user's input.
- Inserting the search term into the SQL query.
- Executing the query and retrieving the results.
- Formatting and displaying the results on the webpage.



&nbsp;


## 🛒 Add Product to Customer's Order (Add to Cart)

When a customer clicks **"Buy Now"**, the online store needs to insert the selected product and quantity into the `OrderItem` table.



### 🔑 Key Concepts

- ✅ **Assumption**: An `Order` already exists for the current customer.
- ✅ **Source**: Product details come from the `Product` table.
- ✅ **Target**: Data is inserted into the `OrderItem` table.


## 🧩 Where Do the Values Come From?

| Field in `OrderItem`    | Source Description                                                                 |
|--------------------------|-------------------------------------------------------------------------------------|
| `OrdItemOrdID`           | Provided by the application — the customer's current **Order ID**.                |
| `OrdItemProdID`          | Taken from the **product ID** passed through a hidden form field (`form.prodid`). |
| `OrdItemQty`             | Quantity selected by the customer on the web form (`form.qty`).                   |
| `OrdItemPrice`           | Fetched directly from the `Product` table (`ProdPrice`).                          |
| `OrdItemDiscount`        | Fetched from the product (`ProdDiscount`).                                        |
| `OrdItemTax`             | Usually calculated based on price and customer's location.                        |
| `OrdItemShipping`        | Taken from the product's standard shipping cost (`ProdShipCost`).                 |

&nbsp;

```sql
INSERT INTO Orderitem
OrditemOrdiD
, OrditemProdiD
, OrditemQty
, OrditemPrice
, OrditemDiscount
, OrditemTax
, OrditemShipping
SELECT OrdiD
, ProdiD
, form.qty
, ProdPrice
, ProdDiscount
, calculation
, ProdShipCost
FROM Product
WHERE ProdiD = •form.prodid'
```


> [!NOTE]
> - `INSERT ... SELECT` is a useful technique to **pull data from another table (Product)** and **insert it into OrderItem** in one operation.
> - The **current Order ID** must be dynamically inserted into the query — it's managed by the application session.
> - Tax is often **calculated** at runtime based on business logic, such as location and tax rules.
> - This approach avoids redundant manual input and ensures price consistency with current product data.



> ⚠️ This is a simplified example meant to show how data flows from the product catalog into the customer’s current order.


