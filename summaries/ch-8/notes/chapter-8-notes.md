# 📘 Advanced Database Features

This chapter covers important features used to write advanced queries, organize data with views, extend logic through procedures and triggers, and ensure performance and data consistency also covers the concurancy and database tuning.

## ✅ Content

 - [1. Advanced Quaries](/summaries/ch-8/notes/chapter-8-notes.md#1️⃣-advanced-quaries)
 - [2. Stored Procedures and Triggers](/summaries/ch-8/notes/chapter-8-notes.md#2️⃣-stored-procedures-and-triggers)
 - [3. Physical Database Tuning](/summaries/ch-8/notes/chapter-8-notes.md#3️⃣-physical-database-tuning)
 - [4. Indexes](/summaries/ch-8/notes/chapter-8-notes.md#4️⃣-understanding-indexes-a-practical-view)
 - [5. Managing Concurrency](/summaries/ch-8/notes/chapter-8-notes.md#5️⃣-managing-concurrency-in-databases)

---

# 1️⃣ Advanced Quaries
Why Use Advanced Queries?

- Perform more **sophisticated filtering** than simple WHERE clauses
- Reduce **application-side processing** by letting the database engine do the work
- Improve **performance** and reduce **network traffic**


## 🔄 Sub-Queries
Sub-queries are `SELECT` statements **nested inside** other SQL queries. They’re used when you want to use the result of one query **inside** another — often in the `WHERE` or `HAVING` clause.

### 📌 Use Case: Products Above Average Price
Get all products priced above the average product price from this table:

![Product Table.](/summaries/ch-8/imgs/product-table.jpg)


```sql
SELECT Products.ProductShortName, Products.ProductPrice
FROM Products
WHERE ProductPrice > (SELECT AVG(ProductPrice) FROM Products)
```

> The inner query (SELECT AVG(ProductPrice) FROM Products) calculates the average, and the outer query retrieves products that exceed it.



> [!NOTE]
> - Some databases support multi-level nesting of sub-queries; others have limitations.
> - Always test performance for sub-queries, especially on large datasets — what is logically elegant may not always be optimized.



&nbsp;

### 📌 Use Case: Categories With Higher Avg Price
Return categories whose average product price is greater than the overall average:

```sql
SELECT Products.CategoryID, AVG(Products.ProductPrice) AS AvgOfProductPrice
FROM Products
GROUP BY Products.CategoryID
HAVING AVG(Products.ProductPrice) > (SELECT AVG(ProductPrice) FROM Products)
```

> This query selects all categories where the average product price is above the overall average.



> [!TIP]
> Always aim to perform data manipulation at the database level to benefit from query optimization, indexing, and lower network load.




## 🔄 Sub-Query Set Operators

Sub-queries are not limited to simple comparison operators like `>` — they can also be used with **set operators** to perform comparisons involving multiple values.

&nbsp;

### 🧠 Use Case: Selecting Managers from a Recursive Relationship

Suppose you want to identify all employees who **manage others** in the organization, in order to send them on an HR training seminar. The data uses a **recursive relationship** where each employee's `Manager` field refers to another employee's `EmployeeID`.

![Manager Relationship Table](/summaries/ch-8/imgs/manager-relationship-table.jpg)

To find all managers, we need employees whose:

- `Manager` is `NULL` (top of the hierarchy), or
- `EmployeeID` appears in another employee’s `Manager` column.

A naive approach might look like:

```sql
SELECT Employees.EmployeeName  
FROM Employees  
WHERE (Manager IS NULL) OR (EmployeeID = 1) OR (EmployeeID = 2);
```

> This works but is **hard-coded** and must be updated whenever new managers are added.

&nbsp;


### ✅ Using the `IN` Operator

We can use the `IN` operator to simplify multiple OR checks:

```sql
 SELECT Employees.EmployeeName  
 FROM Employees  
 WHERE (Manager IS NULL) OR (EmployeeID IN (1, 2));
```

> Still, this example uses hardcoded values. To solve this, we can **combine `IN` with a sub-query** to make it dynamic:

```sql
 SELECT Employees.EmployeeName  
 FROM Employees  
 WHERE (Manager IS NULL)  
 OR (EmployeeID IN (SELECT DISTINCT Manager FROM Employees));
```


> ✔️ This version is **dynamic**, **readable**, and **future-proof** — it adjusts automatically as managers change.

&nbsp;


### 🔁 Could We Use a Self-Join?

Yes — a **self join** would also solve this, similar to recursive joins in Chapter 4. However, sub-queries are often more readable and simpler to maintain for this use case.

&nbsp;


## 🔧 Set Operators Supported in Sub-Queries

Here are the four main set operators you can use in sub-queries:

- `IN` — check if a value exists in a list
- `ALL` — compare against **all** values returned by the sub-query
- `ANY` — compare against **any** value returned by the sub-query
- `EXISTS` — check if the sub-query returns **at least one row**

&nbsp;

## 🔁 Correlated Sub-Queries

Correlated sub-queries allow the inner sub-query to reference a value from the **outer query**. Unlike regular sub-queries, the inner query in a correlated sub-query is **re-evaluated for each row** returned by the outer query.

&nbsp;


### 🧠 Use Case: Products with Above-Average Price in Their Category

Let's say we want to list all products whose price is higher than the **average price in their own category**. We achieve this using a correlated sub-query, as shown below:

```sql
 SELECT ProductsOuter.ProductShortName, ProductsOuter.ProductPrice  
 FROM Products ProductsOuter  
 WHERE ProductsOuter.ProductPrice >  
 (SELECT AVG(ProductPrice) FROM Products  
 WHERE Products.CategoryID = ProductsOuter.CategoryID);
```

> This query compares each product’s price to the **average price of other products in the same category**.

&nbsp;


### 🔍 How It Works

- We give the outer query's `Products` table an alias: `ProductsOuter`.
- The inner sub-query dynamically recalculates the category average **for each product** using the current `CategoryID` from the outer query.
- The outer query then checks if the product's price is greater than its category average.

&nbsp;


### 💡 Why Use an Alias?

We’re referencing the **same table** twice — once in the outer and once in the inner query — so we use an alias to distinguish between them.

&nbsp;


### 🧮 Example Walkthrough

Suppose this is our `Products` table:

![Products Table](/summaries/ch-8/imgs/product-table-rows.jpg)

For each row, the database:

1. Retrieves that row in the outer query.
2. Runs the sub-query for that row’s `CategoryID`.
3. Compares the product's price to the result of the sub-query.

Example for row 5 (Product in `CategoryID = 6`):

```sql
 SELECT AVG(ProductPrice) FROM Products  
 WHERE Products.CategoryID = 6;
```

> If the result is `$105` and the product price is `$150`, then `$150 > $105`, so it **will be included**.

&nbsp;


### ✅ Final Result

Only the product(s) whose price exceeds their category's average will be returned. In this case:

![Result Set](/summaries/ch-8/imgs/prdocuts-avg-result-set.jpg)

> Correlated sub-queries are powerful for row-by-row comparisons against dynamically computed values.

&nbsp; 


## 🔎 Sub-Query Operators: IN, EXISTS, ANY, ALL

Advanced SQL allows us to perform **set-based filtering** using sub-query operators. These operators — `IN`, `EXISTS`, `ANY`, and `ALL` — give us powerful ways to compare data between multiple sets.

&nbsp; 

### 🧩 `IN` Operator

The `IN` operator checks if a value **exists within a list or sub-query result**. It returns `TRUE` if the value is found and `FALSE` otherwise.

Example with hardcoded list:


```sql
 SELECT Employees.EmployeeName  
 FROM Employees  
 WHERE EmployeeID IN (1, 2);
```

Example using a sub-query to avoid hardcoding:


```sql
 SELECT Employees.EmployeeName  
 FROM Employees  
 WHERE EmployeeID IN (SELECT DISTINCT Manager FROM Employees);
```


> [!TIP]
> When building web forms with checkboxes for filtering categories, checkbox values (usually sent as comma-separated values) can be inserted directly into an `IN` clause to dynamically generate the query.

&nbsp; 

## 📥 `EXISTS` Operator

The `EXISTS` operator checks **whether a sub-query returns any rows**. It's always used with **correlated sub-queries** and is typically more efficient due to index optimization.

Example (Get employees who are managers):

```sql
 SELECT EmployeesOuter.EmployeeName  
 FROM Employees EmployeesOuter  
 WHERE Manager IS NULL  
 OR EXISTS (SELECT * FROM Employees  
 WHERE Employees.Manager = EmployeesOuter.EmployeeID);
```

>  `EXISTS` is optimized to quickly test the presence of matching rows, not to retrieve them.

&nbsp; 


## 🔁 `ANY` and `ALL` Operators

These are used with scalar comparison operators (`=`, `<`, `>`, etc.) to compare a **single value to a set**.

#### 🔹 `ANY`

Returns `TRUE` if **any** comparison to items in the list is true.

Examples:
> 3 > ANY (1, 2, 3) → `TRUE`  
> 0 > ANY (1, 2, 3) → `FALSE`

#### 🔹 `ALL`

Returns `TRUE` if **all** comparisons to items in the list are true.

Examples:
> 2 > ALL (1, 2, 3) → `FALSE`  
> 4 > ALL (1, 2, 3) → `TRUE`

#### 🧪 Real Example:
Get products whose price is greater than **all** prices in Category 1:


```sql
 SELECT Products.ProductShortName, Products.ProductPrice  
 FROM Products  
 WHERE ProductPrice > ALL (  
 SELECT ProductPrice FROM Products  
 WHERE CategoryID = 1  
 );
```
&nbsp; 


These operators enable **powerful filtering logic** in SQL, especially useful when working with dynamically generated sets or when optimizing performance with indexed sub-queries.



## 📄 Views

A **view** is a **virtual table** based on a `SELECT` statement. It does not store data itself but presents data from other tables as if it were a real table. Views simplify complex queries, enhance security, and improve performance through query plan reuse.

&nbsp; 


### 🔧 Creating a View

Example:


```sql
 CREATE VIEW CategoriesAndSummary  
 AS  
 SELECT Categories.CategoryID,  
        Categories.CategoryName,  
        Avg(Products.ProductPrice) AS AveragePrice  
 FROM Categories  
 INNER JOIN Products ON Categories.CategoryID = Products.CategoryID  
 GROUP BY Categories.CategoryID, Categories.CategoryName;
```

> Once created, you can query it like a regular table:

```sql
 SELECT * FROM CategoriesAndSummary;
```

![Categories average priceses](/summaries/ch-8/imgs/categories-avergae-price.jpg)


&nbsp; 


### 💡 Benefits of Using Views


> - **Simplifies complex joins or aggregate queries**: Instead of repeating the logic in every query, wrap it in a view.
> - **Improves performance**: Views allow the database to compile and reuse the query plan.
> - **Enhances security**: You can expose only a subset of columns (e.g., hiding salaries or personal info) via the view while restricting access to the base table.
> - **Supports chaining**: Views can be built on top of other views for more modular and layered design.

&nbsp; 


### 🔒 Example: Data Access Control

Let’s say the `Employees` table contains sensitive information. Instead of giving direct access, create a restricted view:

```sql
 CREATE VIEW BasicEmployees AS  
 SELECT EmployeeID, FirstName, LastName, Department  
 FROM Employees;
```

> This allows access to essential details only — excluding fields like salary or personal contact info.

✅ Views provide an **elegant mechanism** for reusing logic, enforcing data access policies, and optimizing performance in your SQL-based applications.

---

# 2️⃣ Stored Procedures and Triggers

Most modern relational databases support **stored procedures** and **triggers**, which enable procedural logic to run inside the database itself.

> [!NOTE]
> Unlike SQL syntax for queries, **stored procedure syntax is database-specific**. Examples in this section use **Transact-SQL (T-SQL)** for Microsoft SQL Server.

&nbsp; 

## 🛠️ Stored Procedures

A **stored procedure** (also known as *sprocs*) is a precompiled collection of SQL statements and optional control-of-flow logic. It can accept input parameters, return output parameters, and be executed multiple times efficiently.



### ⚡ Performance Benefits

- Stored procedures are **compiled and cached** by the database engine.
- When reused, they **bypass the parsing and planning stage**, unlike ad-hoc SQL strings.
- Execution is **faster and more consistent**, especially when using multiple queries within a single call.
- Server-side execution reduces network traffic and enhances security.

> ✅ First-time execution: parsed, optimized, and cached  
> ✅ Repeated execution: uses precompiled query plan  
> ❌ Ad-hoc SQL: parsed and planned every time

&nbsp; 

### 🛡️ Business Logic & Security

- Encapsulating business rules within stored procedures prevents inconsistency across applications.
- Helps prevent accidental misuse by developers or clients.
- You can **restrict access to tables** and expose **only stored procedures** to control how the data is accessed or modified.


#### 📌 Example: Insert with Validation

```sql
 CREATE PROCEDURE AddOrganization  
 (  
   @Name varchar(50),  
   @Email varchar(50)  
 )  
 AS  
 IF EXISTS (SELECT * FROM Organization WHERE Name = @Name)  
   RETURN  
 INSERT INTO Organization (Name, Email)  
 VALUES (@Name, @Email)
```

This procedure:
- Accepts `@Name` and `@Email` as inputs.
- Checks if the organization already exists.
- Inserts a new row only if it does not.

&nbsp; 


## 🔁 Output Parameters

Stored procedures can return values using **OUTPUT parameters**, allowing the caller to retrieve processed data.

### 📌 Example: Get Average Product Price by Category

```sql
 CREATE PROCEDURE GetAvgPrice  
 (  
   @CategoryID int,  
   @AvgPrice money OUTPUT  
 )  
 AS  
 SELECT @AvgPrice = AVG(ProductPrice)  
 FROM Products  
 WHERE CategoryID = @CategoryID
```

This procedure:
- Accepts a `CategoryID`.
- Calculates and returns the average price using the `@AvgPrice` output parameter.

&nbsp; 



### 📌 Use Cases from Real Projects

#### 🛍️ Inserting Orders in an E-Commerce System

> Instead of sending separate INSERT queries for each order item, a stored procedure was used to receive the entire order in XML format.  
> The procedure parsed the XML and inserted all data in one transaction — faster, safer, and less prone to failure.

#### 🔔 Alarm Detection in Time Series Data

> A stored procedure was used to detect alarm thresholds in river flow data (30,000+ rows/site).  
> Doing this in application code would overload the web server.  
> Instead, a stored procedure evaluated complex conditions and returned a simple True/False.

#### 🌐 Remote Database Interaction

> In a distributed system over the internet, stored procedures were used to minimize data transferred.  
> Only essential input and output were passed, reducing network load by ~80%.

&nbsp; 


> [!NOTE]
> Stored procedures are essential for:
> - Reducing code duplication
> - Improving performance
> - Centralizing business logic
> - Enabling secure and scalable data access




## 🔄 Triggers 

Triggers are special types of stored procedures that are automatically executed in response to specific events on a table — such as INSERT, UPDATE, or DELETE.


Stored procedures and triggers provide **modularity**, **performance**, and **centralized business rules**, and can significantly reduce application-side complexity.



### ⚙️ What Do Triggers Do?

- Automatically respond to data changes (e.g., `insert a row`, `check constraints`, `notify users`).
- Can automate logic such as:
  - Logging changes
  - Sending notifications
  - Enforcing business rules

&nbsp; 


### 🧠 Types of Triggers

- **AFTER Triggers**: Execute **after** the event occurs.
- **INSTEAD OF Triggers**: Replace the default behavior of `INSERT`, `DELETE`, or `UPDATE`.
  - Useful when you want to block actual deletes and instead set a `Deleted` flag.

> Example: Prevent deleting products by using a trigger that sets a `Deleted = True` flag instead of actually removing the row.

&nbsp; 


### ✅ Virtual Tables

Triggers have access to two **virtual tables**:
- `inserted`: contains new rows
- `deleted`: contains old rows

These are read-only and only exist during the execution of the trigger.


 **Sample Trigger:** 
```sql
 CREATE TRIGGER CreateObjects  
 ON SiteMeasurements  
 FOR INSERT  
 AS  
 DECLARE @ItemNumber AS Varchar(20)  
 SELECT @ItemNumber = inserted.SiteMeasurementID FROM inserted  
 EXEC('CREATE TABLE dbo.DataTable' + @ItemNumber + ' (  
   [DateStamp] [datetime] NOT NULL,  
   [DataValue] [float] NOT NULL,  
   CONSTRAINT [DataTablePrototype' + @ItemNumber + '_PK] PRIMARY KEY CLUSTERED ([DateStamp])  
 )')
```


> [!WARNING]
> - Be careful of **circular triggers** (e.g., Table A trigger inserts into Table B, which triggers back into Table A).
> - Most RDBMSs **block cascading triggers** by default to prevent infinite loops.
> - Use **stored procedures** for more controlled cascading logic. 

&nbsp; 


### 📌 Real-World Use Cases

#### 📈 Dynamic Table Creation
> Created time series tables dynamically for real-time web-based graphing (30,000+ rows per site).  
> Triggers generated new tables upon data insertion.

#### 🛒 Content Aggregation
> E-commerce mall used triggers to sync product data from multiple store databases into a centralized "mall" database.  
> Changes in a store’s product table automatically reflected in the mall’s global product catalog.

#### 📝 Audit Trails
> Legal/accounting document portal tracked all deletions and updates.  
> Used triggers to copy old data into an audit table with timestamp metadata.

#### 🧮 Summary Data in Denormalized Columns
> For faster querying, product purchase counts were stored in a denormalized column.  
> ON INSERT triggers updated this column when a new order was placed.

---


# 3️⃣ Physical Database Tuning

In earlier chapters, we focused on the **logical** design of a database, abstracting away the physical details. This abstraction lets us focus on solving problems without worrying about how the database stores data on disk. However, there are times when the default physical implementation chosen by the DBMS isn't the most efficient.

> In some situations—especially with large databases or high-traffic systems—optimizing the physical layer can result in significant performance improvements.

&nbsp; 


## 💡 Logical vs Physical Model

- **Logical Model**: The schema and structure we define as database designers.
- **Physical Model**: How the DBMS actually stores, retrieves, and manages that data under the hood.

Most of the time, the DBMS makes reasonable assumptions about storage and access patterns. But for critical systems, those assumptions may need to be overridden for better performance.

&nbsp; 


## 🧠 How the DBMS Executes Queries

When a query is sent to the DBMS:

1. It breaks the query into an internal format.
2. It evaluates multiple possible execution strategies.
3. It selects the **lowest-cost** option based on system resources.



## 🧪 Query Plan Analysis

To help understand or influence how a query will be executed, most RDBMSs provide two important tools:

- **EXPLAIN (or equivalent)**:  
  Ask the DBMS to show its planned strategy for executing a query.
  
- **HINTS**:  
  Suggest (or enforce) a preferred execution method (e.g., which index to use).

>[!CUATION]
> Don't optimize queries too early. Query optimizations only make sense when:
> - You have a **realistic volume of data**.
> - The system is **already operational**.
> - Performance bottlenecks have been observed.

&nbsp;


>[!TIP]
> 🔧 When Should You Physically Tune a Database?
> - When performance issues are detected under **production-like load**.
> - When dealing with **large datasets**.
> - When the **query plan** shows inefficient strategies.
> - When application logic requires **very fast access** to critical tables.

&nbsp;


## 🔍 Explaining Queries

Before optimizing how the RDBMS physically retrieves data, we first need to understand **how** it currently does so. Most RDBMS platforms provide an **EXPLAIN** utility to analyze the **query execution plan**.

> These plans help reveal which indexes are used, the join strategy, and the order of operations. This allows you to make informed decisions about indexing and query structure.

&nbsp;

### 🧾 Platform-Specific EXPLAIN Commands

Each RDBMS uses slightly different syntax for its EXPLAIN functionality. Refer to your DBMS documentation for details.

![Explain commands table](/summaries/ch-8/imgs/explain-commands-table.jpg)


&nbsp;

### 🧪 Example: SQL Server Query Plan

Let's analyze this query using the `Products` table from the Northwind database:

![Categories average priceses](/summaries/ch-8/imgs/ms-northwind-products.jpg)

```sql
 SELECT Products.*  
 FROM Products  
 WHERE UnitPrice > (SELECT Avg(UnitPrice) FROM Products);
```

In SQL Server, we can view the query plan using **SHOWPLAN**. The output includes steps like:

- Clustered Index Scan
- Compute Scalar
- Nested Loops
- Stream Aggregate

![query paln before index](/summaries/ch-8/imgs/query-pan.jpg)


Each step has a **percentage cost** assigned. These indicate the **relative cost** of each operation. Hovering over icons in the graphical plan view shows more information.

&nbsp;

![clustred index scan](/summaries/ch-8/imgs/clustred-index-scan.jpg)


In this example, the sub-query calculating `Avg(UnitPrice)` requires a **clustered index scan**, which is expensive.

> 🟡 The root issue: There's no index on the `UnitPrice` column, so SQL Server must scan the entire table.

&nbsp;

### 📉 Fixing with Indexes

If we create an index on the `UnitPrice` column, we can significantly reduce the cost of this operation:

![query paln after index](/summaries/ch-8/imgs/query-plan-after-index.jpg)


The optimizer can now perform a **clustered index seek** rather than a scan. This shifts the cost away from parts of the query that can be optimized.

> ⚠️ Note: The total cost is always **100%**. Optimizing just redistributes where the cost occurs to minimize **overall execution time**.

&nbsp;

## 💡 Using Hints

In some situations, an index **exists** but the RDBMS **chooses not to use it**. This is where **query hints** can help.

> Hints are directives inserted into your SQL that guide the optimizer on how to execute the query.

![hints](/summaries/ch-8/imgs/hints.jpg)

Example (SQL Server):

```sql
 SELECT Products.*  
 FROM Products  
 WHERE UnitPrice > (SELECT Avg(UnitPrice) FROM Products)  
 OPTION (FAST 5);
```

This hint tells SQL Server to **optimize for returning 5 rows quickly**, even if it sacrifices performance for additional rows.

&nbsp;



> [!HINT]
> - Always **benchmark** and **profile** before applying hints.
> - Re-profile as your data grows—hints that help today may **hurt** performance tomorrow.

---

# 4️⃣ Understanding Indexes (A Practical View)

Indexes are like the **card catalogs in a library**. They help us **locate data faster** by pointing the database directly to where a record lives on disk—rather than scanning every row one by one.


### 📖 The Library Analogy

Just like a library can have indexes by:
- **Author**
- **Title**
- **Subject**

A **database** can have indexes on:
- **One or more columns**
- **Primary and foreign keys**
- **Frequently searched fields**

Without an index, a **full table scan** must occur—like searching every shelf in the library for a red book. 

&nbsp;

## 🔄 Indexing Is a Trade-Off

> More indexes = Faster SELECTs  
> Fewer indexes = Faster INSERTs/UPDATEs

### 🏗️ The Cost of Indexing

- Every **new record** requires index maintenance
- Frequent changes can cause **index fragmentation**
- Index files consume **disk space**
- Too many indexes = slower **write operations**

This is why **blindly indexing everything is a bad idea**.

&nbsp;

## ⚠ When Should You Index?

> ✅ Always Index Primary and Foreign Keys
> - Most RDBMSs do this automatically.
> - They're frequently used in **joins** and **lookups**.

> ✅ Index Columns Used in WHERE Clauses (on read-heavy data)
> - Especially for **public data** accessed by site visitors
> - Example: product listings, categories, blog posts

> [!WARNING]
> ⚠️ Don’t index rarely used filters or private/admin-only fields unless necessary.

> [!CAUTION]
> Avoid Indexing Write-Heavy Tables
> - For tables with many **INSERTs**, like visitor logs or click tracking
> - Tip: Use **denormalized, unindexed tables** for logging
    - Later process them into a separate, optimized structure for analysis.

&nbsp;

## 🌐 Web Developer Best Practices

| Scenario | Indexing Recommendation |
|----------|--------------------------|
| Public product catalog (read-heavy) | ✅ Index all frequently searched fields |
| User activity log (write-heavy) | ❌ Avoid indexing or keep to a minimum |
| Admin dashboard with occasional updates | ⚠️ Light indexing on filtered fields |


&nbsp;
# 📌 Non-Clustered vs. Clustered Indexes

Understanding how different indexes work is **crucial** for designing fast and efficient databases—especially as your data grows.



## 🔎 Non-Clustered Indexes

A **non-clustered index** is **separate** from the data in the table. It works like a **lookup table**:

- Contains keys (based on columns you choose)
- Each key points to the **row location** in the table
- Can be **multiple** per table
- Can be **unique** or **non-unique**

### ✅ Use When:
- You want to speed up **lookups** on specific columns
- You don’t need data to be **physically sorted**
- You need **multiple indexes** on different columns

### ⚠️ Downsides:
- Slows down **INSERT/UPDATE/DELETE**
- Takes **extra disk space**
- Too many = **maintenance overhead**

&nbsp;

## 🧱 Clustered (Physical) Indexes

A **clustered index** stores the **actual data rows in sorted order** based on the index key.

> Think of it like sorting books **on the shelf** directly, not just on the index cards.

- Only **one per table**
- Data is **physically sorted**
- Used **automatically** in some RDBMSs (e.g., primary key in SQL Server)

### ✅ Use When:
- You query **ranges of data** (e.g., dates, prices)
- You want to **read sequentially** from disk
- Data is **rarely updated**

### ⚠️ Downsides:
- Slows down **INSERTs** (must maintain physical order)
- **Expensive to change** (reordering table)
- Choose **wisely**, since there can only be one

&nbsp;

## 📈 Example: Why Clustered Index Helps with Ranges

Suppose we store **time series** data with one row every 15 minutes:

```sql
SELECT DateStamp, DataValue
FROM DataTable131
WHERE DateStamp BETWEEN GETDATE()-90 AND GETDATE()-60;
```

If **DateStamp** is:
- 🛠 **Non-clustered**: DB finds row in index, then jumps to its disk location (slow disk seeks)
- 🚀 **Clustered**: DB finds start row and **reads sequentially** to end of range (much faster)

&nbsp;

## 🧠 Key Differences Summary

| Feature                  | Non-Clustered Index         | Clustered Index (Physical)    |
|--------------------------|-----------------------------|-------------------------------|
| Storage                  | Separate from table         | Part of the table             |
| Data Order               | Unsorted                    | Sorted based on index key     |
| Number per table         | Multiple allowed            | Only one allowed              |
| Lookup Speed             | Fast                        | Fast (especially for ranges)  |
| Write Performance        | Slower as more indexes      | Slower (reordering needed)    |
| Disk Access Pattern      | Random seeks                | Sequential access             |

&nbsp;

## 🧠 Choosing Wisely

- 🔑 Use **clustered index** on most frequently range-filtered column (e.g., DateStamp).
- 🔍 Use **non-clustered indexes** for fast lookup on foreign keys, names, status columns, etc.
- ⚖️ Balance your **read vs. write performance** needs.

> 💡 You can often combine a **clustered index** on time-based data with **non-clustered** indexes on category, region, or status to optimize for both analytics and reporting.

---

# 5️⃣ Managing Concurrency in Databases

In a **web-based environment**, multiple users may try to access or modify the database **simultaneously**. Managing this **concurrent access** is critical to ensure:

- ⚖️ **Data consistency**
- 🧱 **System integrity**
- 🚀 **Performance**



## 🔐 Locking: The Basics

**Locking** places a *temporary flag* on a data object to indicate it’s being used, restricting access by others.

### 🔑 Lock Granularity Levels:
| Level     | Description                     |
|-----------|---------------------------------|
| Field     | Individual column in a row      |
| Row       | Specific record in a table      |
| Table     | Whole table                     |
| Database  | Entire database                 |

> 🧠 Not all databases support fine-grained locking (e.g., row-level).

### 🔁 Lock Types:
| Lock Type   | Use Case                          | Allows Concurrent Access? |
|-------------|-----------------------------------|----------------------------|
| Shared      | For reading data (SELECT)         | ✅ Yes, unless there's an exclusive lock |
| Exclusive   | For writing data (INSERT/UPDATE)  | ❌ No, blocks other locks |

&nbsp;

## 🔄 Transactions: All or Nothing

A **transaction** is a **logical unit of work** that groups multiple SQL operations together. It **guarantees** that either *all operations succeed* or *none do*.

### 📦 Why Use Transactions?
Imagine an e-commerce site inserting:
- 1 row into `Orders`
- Multiple rows into `OrderItems`

What if the server crashes after inserting the order but before inserting items?  
👉 You get **incomplete** or **corrupt** data.

### ✅ Solution: Transactions

```sql
BEGIN TRY
    BEGIN TRANSACTION;
    -- Insert order
    INSERT INTO Orders (...) VALUES (...);

    -- Insert order items
    INSERT INTO OrderItems (...) VALUES (...);
    INSERT INTO OrderItems (...) VALUES (...);
    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    PRINT ERROR_MESSAGE();
END CATCH;
```

> [!NOTE]
> This will **undo all changes** made within the transaction if it fails.

&nbsp;

# 🧩 Locking & Transactions Work Together

- The **RDBMS** automatically sets locks based on the **transaction’s isolation level**
- Developers **don't usually need to set locks manually**
- Proper use of **transactions** ensures **data integrity**, especially in **multi-user environments**


# 🔐 Isolation Levels 

Isolation levels determine **how visible** one user’s changes are to others during a transaction:


## 🔹 1. `READ UNCOMMITTED`

> The lowest isolation level. Allows reading uncommitted (dirty) changes made by other transactions.

#### 🔧 How It Works
- No locks are acquired when reading data.
- Can read rows being modified by other uncommitted transactions.

#### ✅ Advantages
-  Fastest performance.
-  Useful for analytics or monitoring where exact accuracy is not required.

#### ❌ Disadvantages
-  Dirty Reads: You may read data that will be rolled back.
-  Inconsistent data and unexpected results.
-  Violates ACID consistency.

### 🧪 Example


```sql
-- Transaction A
BEGIN TRANSACTION;
UPDATE Products SET Price = 50 WHERE ProductID = 1;
-- Not committed yet

-- Transaction B
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT Price FROM Products WHERE ProductID = 1;
-- Will read the uncommitted price (50)
```


&nbsp;

## 🔸 2. `READ COMMITTED` *(Default in most RDBMSs)*

> Prevents dirty reads. Only committed data can be read.

### 🔧 How It Works
- A shared lock is placed on rows while reading.
- Waits for uncommitted transactions to complete before reading.

### ✅ Advantages
-  No dirty reads.
-  Good balance between performance and consistency.

### ❌ Disadvantages
-  Non-repeatable reads allowed.
-  Phantom rows may appear.

### 🧪 Example

```sql
-- Transaction A
BEGIN TRANSACTION;
SELECT Balance FROM Accounts WHERE ID = 1; -- Reads $1000

-- Transaction B (in between A's statements)
BEGIN TRANSACTION;
UPDATE Accounts SET Balance = Balance - 100 WHERE ID = 1;
COMMIT;

-- Transaction A again
SELECT Balance FROM Accounts WHERE ID = 1; -- Might now read $900
```




## 🔷 3. `REPEATABLE READ`

> Prevents dirty reads and non-repeatable reads. But phantom rows are still possible.

### 🔧 How It Works
- Shared locks are held on all rows read for the duration of the transaction.
- Prevents updates/deletes of those rows by other transactions.

### ✅ Advantages
-  Guarantees the same row values throughout the transaction.
-  Prevents inconsistent reads.

### ❌ Disadvantages
-  Phantom rows may appear (new rows can still be inserted).
-  May block other writers for longer periods.

### 🧪 Example


```sql
-- Transaction A
BEGIN TRANSACTION;
SELECT * FROM Orders WHERE CustomerID = 1; -- Returns 5 rows

-- Transaction B
INSERT INTO Orders (CustomerID, ...) VALUES (1, ...);
COMMIT;

-- Transaction A again
SELECT * FROM Orders WHERE CustomerID = 1; -- Might return 6 rows
```


&nbsp;

## 🔶 4. `SERIALIZABLE`

> Highest isolation level. Emulates serial execution of transactions.

### 🔧 How It Works
- Locks entire ranges of data or even table scans.
- Prevents dirty reads, non-repeatable reads, and phantom reads.

### ✅ Advantages
-  Maximum data consistency.
-  Ensures transaction isolation like serialized access.

### ❌ Disadvantages
-  Slowest – may block many transactions.
-  High chance of deadlocks and contention.

### 🧪 Example

```sql
-- Transaction A
BEGIN TRANSACTION;
SELECT COUNT(*) FROM Inventory WHERE ProductType = 'Books';

-- Transaction B (attempts this during A)
INSERT INTO Inventory (ProductType) VALUES ('Books');
-- Blocked until A commits or rolls back
```

&nbsp;



| Level              | Prevents...              | Allows...                          |
|--------------------|--------------------------|------------------------------------|
| Read Uncommitted   | Nothing                  | Dirty reads                        |
| Read Committed     | Dirty reads              | Non-repeatable reads               |
| Repeatable Read    | Non-repeatable reads     | Phantom reads                      |
| Serializable       | Phantom reads            | Full isolation, least concurrency  |

> ⚠️ Higher isolation = More locks = Less concurrency

&nbsp;

## 🧪 Real-World Examples

### ✔️ Safe Order Insertion:
Wrap inserting into `Orders` and `OrderItems` in a **transaction**.

### ✔️ Banking Transfers:
Subtract from one account and add to another **in the same transaction**.

### ✔️ Batch Operations:
Bulk updates or deletes wrapped in a transaction to allow **rollback on failure**.

&nbsp;


## 🧵 Summary

| Feature        | Purpose                            |
|----------------|-------------------------------------|
| **Locking**     | Prevents conflicting data access   |
| **Transactions**| Groups operations into one logical unit |
| **Commit**      | Finalizes all changes              |
| **Rollback**    | Cancels and undoes all changes     |

Proper **concurrency management** ensures your database handles **high traffic safely** without corrupting data or reducing performance.

&nbsp;

# 💡 ACID Properties of Database Transactions

The **ACID** acronym defines the key properties of a reliable transaction in any RDBMS.

| ACID Property | Description |
|---------------|-------------|
| **A** — Atomicity   | All or nothing |
| **C** — Consistency | Valid state transitions |
| **I** — Isolation    | No side effects from parallel transactions |
| **D** — Durability   | Once committed, it's permanent |

&nbsp;


## 🔹 A — Atomicity

> A transaction must **either complete entirely or have no effect at all**.

This ensures that operations within a transaction are treated as a single unit. If one part fails, the whole transaction is rolled back.

### 🧪 Example:

```sql
BEGIN TRANSACTION;

-- Deduct money from user
UPDATE Accounts SET Balance = Balance - 100 WHERE AccountID = 1;

-- Add money to recipient
UPDATE Accounts SET Balance = Balance + 100 WHERE AccountID = 2;

-- If something fails here, rollback everything
COMMIT;
```

If the second `UPDATE` fails, the first one is **also undone**, preserving data integrity.

&nbsp;


## 🔸 C — Consistency

> Transactions must leave the database in a **valid state**, adhering to **rules and constraints**.

Consistency ensures that **business rules and data integrity constraints** are never violated during a transaction.

### 🧪 Example:

Suppose a `Balance` must never go below 0 (business rule). The following transaction would violate that unless it is **rolled back**.

```sql
-- Violates business rule: balance can't go below 0
UPDATE Accounts SET Balance = Balance - 500 WHERE AccountID = 1;
```

The RDBMS (or application logic) should **prevent committing** this transaction if it breaks rules.

&nbsp;


## 🔷 I — Isolation

> The effects of a transaction are **not visible** to other transactions until committed.

This prevents issues like **dirty reads**, **non-repeatable reads**, and **phantom reads** depending on the isolation level.

### 🧪 Example:

```sql
-- Transaction A
BEGIN TRANSACTION;
UPDATE Products SET Stock = Stock - 1 WHERE ProductID = 100;

-- Transaction B (before A commits)
SELECT Stock FROM Products WHERE ProductID = 100;
-- Should not see updated stock if A hasn't committed yet
```

Proper isolation ensures **Transaction B** doesn't see **intermediate or uncommitted changes** from A.

&nbsp;

## 🔶 D — Durability

> Once a transaction is **committed**, it is **permanently stored**, even if the system crashes.

Durability guarantees that **committed changes survive crashes** via write-ahead logs or similar recovery mechanisms.

### 🧪 Example:

```sql
BEGIN TRANSACTION;
INSERT INTO Orders (CustomerID, Date) VALUES (123, GETDATE());
COMMIT;
```

If the database server crashes **after the COMMIT**, the order is **still there** when the DB is restarted.

&nbsp;


## 🧠 Summary Table

| Property     | Guarantee                                | Example Problem Prevented     |
|--------------|-------------------------------------------|-------------------------------|
| Atomicity    | All or none of the operations succeed     | Partial updates (e.g., money deducted but not received) |
| Consistency  | Database remains in a valid state         | Violating constraints like foreign keys or business rules |
| Isolation    | Transactions don’t interfere with each other | Reading uncommitted or dirty data |
| Durability   | Committed data survives crashes           | Lost data after a system reboot |


&nbsp;

# ⚙️ Isolation Levels in Action

The syntax for setting isolation levels may vary depending on your RDBMS or programming framework.



### 📌 SQL Server Example
```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN TRANSACTION;

-- Your SQL commands here

COMMIT;
```



### 💻 ADO (e.g., Classic ASP or VB.NET)
```vb
objConnection.IsolationLevel = adXactRepeatableRead
```

&nbsp;

## 🧠 Choosing the Right Isolation Level

To balance **data integrity** and **performance**, follow this general approach:

1. **Start from the highest level** (`SERIALIZABLE`)  
2. Evaluate the **risks introduced** as you go down each level  
3. **Choose the lowest isolation level that satisfies your data consistency needs**

> ✅ In most web applications, **Read Committed** is a good default choice.

&nbsp;

# 🔄 Deadlocks

### ❓ What is a Deadlock?

A deadlock occurs when **two or more transactions** are each waiting for the other to release a lock, resulting in an infinite wait.

### 🔁 Deadlock Example

```
Transaction A               Transaction B
-------------               -------------
Locks Table 1               Locks Table 2
Wants Table 2               Wants Table 1
```

> 🧱 Neither transaction can proceed, and both are blocked permanently unless handled by the DBMS.


> [!NOTE]
> How RDBMS Handles Deadlocks:
> - The **DBMS automatically detects** deadlocks.
> - It selects a **"victim" transaction** to **ro**
