# Practical Web Database Design

## 📘 Introduction

> **What is a Web Database?**  
A database that is connected to the Web in some way. The main difference is performance expectations—users of websites expect **fast** responses.

This book helps you:
- Understand what databases are and how to design them effectively.
- Ensure fast access and minimal storage.
- Design databases specifically with the web in mind.

Throughout the book:
- We'll use standard SQL wherever possible.
- Example databases: MySQL, Oracle, SQL Server.

This book is for you if:
- You’re new to databases but want to use them on your website.
- You’ve worked with web databases but want to improve performance and design.

---

## 📚 Table of Contents

1. [Chapter 2: Core Database Concepts](#chapter-2-core-database-concepts)
   - [Data Models](#1-data-model)
   - [Relational Model](#2-relational-model-table-row-column)
   - [Domains & Data Types](#3-domains--data-types)
   - [Nulls & Missing Data](#4-missing-data--nulls)
   - [Primary Keys](#5-primary-key)
   - [Foreign Keys](#6-foreign-key)
   - [Normalization](#7-normalization)
   - [Data Integrity](#8-data-integrity)
   - [Metadata & Data Dictionary](#9-metadata--the-data-dictionary)
   - [Physical Data Access](#10-physical-data-access-models)

2. [Chapter 3: Creating and Using Relational Databases with SQL](#chapter-3-creating-and-using-relational-databases-with-sql)
   - [What is SQL?](#what-is-sql)
   - [DDL, DML, DCL](#SQL-Syntax-Components)
   - [Create Table](#creating-table)
   - [SQL-92 Data Types](#sql-data-types)
   - [Constraints (Column & Table)](#constraints)
   - [DROP, ALTER Table](#drop-table--alter-table)
   - [Create & Drop Index](#create--drop-index)
   - [CRUD: INSERT, UPDATE, DELETE](#crud-operations)
   - [WHERE Clause](#-where-in-sql)
   - [SELECT Basics & Best Practices](#-querying-in-sql-select)
   - [JOINS](#joins)
   - [Aggregates & Grouping](#-aggregation-in-sql--grouping)
   - [Reporting](#-reporting-with-sql-key-features)
   - [Extra Notes & Best Practices](#extra-notes--best-practices)

---

## Chapter 2: Core Database Concepts

### 1. Data Model
- **What is it?**
  - A structured method for organizing and manipulating data.
- **Defines:**
  1. **Structure:** Tables, rows, and columns
  2. **Integrity:** Primary keys, constraints
  3. **Operations:** SQL actions

---

### 2. Relational Model
- A logical model using tables (relations) with rows and columns.
- **Relation:** A table made of tuples (rows) and attributes (columns).
- **Table Components:**
  - Name, columns (with domains), unique rows.

---

### 3. Domains & Data Types
- **Data Types:** Define how data is stored (e.g., INT, CHAR, DATE).
- **Domains:** Set of allowed values per column (can be enforced via constraints).

---

### 4. Missing Data: NULLs
- **When?** Unknown or inapplicable values.
- **Notes:**
  - NULL is not 0 or empty.
  - NULL can't be compared.
  - Use default values only when sensible.

---

### 5. Primary Key (PK)
- Unique identifier for each row (one or more columns).
- **Properties:** Not null, not duplicated, immutable.

---

### 6. Foreign Key (FK)
- Column(s) referring to another table's PK.
- Used to **create relationships** between tables.
- Must match an existing PK value or be NULL.

---

### 7. Normalization
- **Why?** To reduce redundancy and anomalies.
- **Forms:**
  - **1NF:** No multi-valued attributes
  - **2NF:** No partial dependency
  - **3NF:** No transitive dependency
  - **4NF:** No multi-valued dependencies

---

### 8. Data Integrity
- **Goal:** Ensure validity and consistency of data.
- **Types:**
  1. **Entity Integrity:** PK must be unique and not null
  2. **Referential Integrity:** FK must refer to valid PK
  3. **Domain Integrity:** Column values must belong to valid set

---

### 9. Metadata & Data Dictionary
- **Metadata:** Data about data (e.g., table structures, types).
- Stored in a **data dictionary**, often in system tables.

---

### 10. Physical Data Access Models
- **Performance Bottleneck:** Disk I/O is slow compared to RAM.
- **Access Types:**
  1. **Sequential Access:** Record-by-record scan
  2. **Indexed Access:** Use B-tree index
  3. **Direct Access (Hashing):** Fastest for exact key lookups
- **Indexing Tips:**
  - Good for large tables, bad for frequent updates
  - Avoid indexing small tables

---



---

## Chapter 3: Creating and Using Relational Databases with SQL

## What is SQL?

- SQL = Structured Query Language.
- Declarative, not procedural.
- Used to retrieve, create, modify, and delete database structures and records.

## SQL in Web

- Web apps don’t run raw SQL; they use **stored procedures**:
  - More secure.
  - Reduce network overhead.

## SQL Syntax Components

### 1. DDL – Data Definition Language

Used to define database schema:

- CREATE TABLE
- DROP TABLE
- ALTER TABLE
- CREATE INDEX
- DROP INDEX

### 2. DML – Data Manipulation Language

Used to work with data:

- SELECT
- INSERT
- UPDATE
- DELETE

### 3. DCL – Data Control Language

Manages access and permissions.

---

## Creating Tables

### Basic Syntax

```sql
CREATE TABLE <table-name> (
  <column-name> <data-type> [DEFAULT <value>] [<column-constraints>],
  ...
  [<table-constraints>]
);
```

- Example:

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

### SQL Data Types

- Numeric: SMALLINT, INTEGER, NUMERIC(p,s), FLOAT
- Date/Time: DATE
- Character: CHAR(n), VARCHAR(n)

---

## Constraints

### Column vs Table Constraints

- Column constraint: applies to one column.
- Table constraint: can apply across multiple columns.

### Common Constraints

- PRIMARY KEY
- FOREIGN KEY
- NOT NULL
- UNIQUE
- CHECK (...)
- CONSTRAINT&#x20;

---

## Modifying Tables

### ALTER TABLE

- Add columns or constraints

```sql
ALTER TABLE EMPLOYEE ADD COLUMN ManagerID INTEGER DEFAULT 0 NOT NULL;
```

- Modify column defaults

```sql
ALTER TABLE EMPLOYEE ALTER COLUMN ManagerID SET DEFAULT 0;
```

- Drop column or constraint

```sql
ALTER TABLE EMPLOYEE DROP COLUMN ManagerID;
```

### DROP TABLE

- Deletes structure and data.

```sql
DROP TABLE CUSTOMER;
```

---

## Indexes

### CREATE INDEX

```sql
CREATE INDEX CustomerNameIndex ON CUSTOMER (Last, First);
CREATE UNIQUE INDEX EmployeeIDIndex ON EMPLOYEE (EmployeeID);
```

### DROP INDEX

```sql
DROP INDEX CustomerNameIndex;
```

---

## INSERT Data

```sql
INSERT INTO EMPLOYEE (EmployeeID, Name, Position, Location, Salary)
VALUES (617258, 'Jane Smith', 'Manager', 'London', 35000);
```

- Can insert NULL or use DEFAULT values.
- Always specify column list for clarity.

---

## UPDATE Data

```sql
UPDATE EMPLOYEE
SET Salary = 37000
WHERE EmployeeID = 617258;
```

- Can update multiple columns.
- Use expressions: `Price = Price * 1.1`
- Use `IS NULL` to handle NULL comparisons.

---

## DELETE Data

```sql
DELETE FROM PRODUCT WHERE ProductNo = 'AAD62726';
```

- Be cautious: DELETE without WHERE deletes all rows.
- Referential integrity may block deletes.

---

## SELECT Queries

### Basic Syntax

```sql
SELECT <columns>
FROM <tables>
[WHERE <conditions>];
```

### Tips

- Avoid `SELECT *`
- Always test WHERE conditions using `SELECT` first.

### Aliases

```sql
SELECT S.SaleNo, C.First FROM SALE AS S JOIN CUSTOMER AS C ON S.CustomerNo = C.CustomerNo;
```

---

## JOINs

- INNER JOIN: shows matched rows only.
- LEFT OUTER JOIN: all rows from left table.
- RIGHT OUTER JOIN: all rows from right table.
- FULL OUTER JOIN: all rows from both.

---

## Aggregation & Grouping

### Aggregates

- COUNT(\*), SUM(col), AVG(col), MIN(col), MAX(col)

### GROUP BY

```sql
SELECT Position, COUNT(*)
FROM EMPLOYEE
GROUP BY Position;
```

- Use HAVING to filter grouped results.

---

## Reporting

### ORDER BY

```sql
SELECT Name, Salary FROM EMPLOYEE ORDER BY Salary DESC;
```

### DISTINCT

```sql
SELECT DISTINCT SaleDate FROM SALE;
```

---

## Real-World Tips

- Use aliases and meaningful column names.
- Always validate your JOIN conditions.
- Test complex WHERE conditions incrementally.
- Consider performance in web apps: pre-compute joins where applicable.


