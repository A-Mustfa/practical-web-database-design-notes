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

---

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

&nbsp;
