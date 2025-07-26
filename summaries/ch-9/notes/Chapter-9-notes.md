
# 📘 Database Maintenance

Database maintenance is the responsibility of the **DBA** (Database Administrator) and includes tasks such as:

- Backing up data
- Reorganizing data & indexes
- Altering the structure of tables
- Optimizing performance

---

## 📑 Chapter Coverage

1️⃣ **Backup and Restore of Database**  
2️⃣ **Modifying Tables**  
3️⃣ **Reorganizing Data & Indexes**  
4️⃣ **Building Scripts for Production Implementation**

---

## ❓ Why Should Developers Know This?

In some environments, **developers** might be responsible for performing database maintenance themselves.

---

## 1️⃣ Backup & Restore

Backup and restore are essential not only for **software or hardware failures** but also for preventing data loss caused by **human mistakes** during development.

---

### ⚠️ Common Mistakes That Require a Backup

1. Forgetting the `WHERE` clause when deleting data.
2. Accidentally deleting the wrong database object.
3. Missing a line of code that leads to **data corruption**.
    

✅ **A good backup plan** with multiple backup points ensures the ability to restore the database to a **stable state**.

---

### 🔹 1.1 Backing Up Your Database

Most RDBMS systems use **two key files**:

1️⃣ **Data file:** contains database objects like tables, views, stored procedures, etc.  
2️⃣ **Log file:** contains all transactions (`INSERT`, `UPDATE`, `DELETE`) + schema modifications.

---

### 🔹 1.2 Types of Recovery Models

#### 🟠 **Simple Recovery Model**

- Restores the database **only** to the last **full backup**.
- Example: If you back up the database every evening and perform changes after the backup, those changes will be lost after restore.
- All **uncommitted transactions** are rolled back.

#### 🟢 **Full Recovery Model**

- Allows recovery to a **specific point in time**.
- Requires **full backups** + **transaction log backups** (e.g., full backup every evening, transaction log every hour).
- During restoration, you apply:
    1. The full backup
    2. The transaction log backups (in sequence) up to the desired time.

---

### 🔹 1.3 Types of Backups

Some RDBMS systems support **differential backups** – only changes made **after** the last full backup are saved.  
When restoring, you **first restore** the full backup **then** the differential backup.

---

#### 📍 **1. Complete Backup**

A **full backup** (data + logs) at a specific point in time.

✅ **SQL Syntax**

```sql
BACKUP DATABASE database_name
TO DISK = 'PATH+FILENAME'
WITH INIT;
-- Overwrites the existing backup file
```

✅ **Example**

```sql
BACKUP DATABASE webDB
TO DISK = 'E:\backup\webDBbak'
WITH INIT;
```

✅ **Stored Procedure for Backup**

```sql
CREATE PROC eComBackUp AS
    DECLARE @emocPath VARCHAR(100)
    SET @emocPath = 'F:\SoftWare\DataBase\BackUps\' 
                    + CAST(DAY(GETDATE()) AS VARCHAR(2)) + '-' 
                    + CAST(MONTH(GETDATE()) AS VARCHAR(2)) + '-' 
                    + CAST(YEAR(GETDATE()) AS VARCHAR(4))

    BACKUP DATABASE [E-Commerce]
    TO DISK = @emocPath
    WITH INIT;
GO

EXEC eComBackUp;
```

---

#### 📍 **2. Transaction Log Backup**

Used when you want to **restore to a specific moment** in time.

✅ **How Full Recovery Model Works:**

1. Create a **full backup** (starting point).
    
2. Create **transaction log backups**.
    
3. Restore **full backup** + each **transaction log backup in order**.
    

---

### ⏰ Timing of Backups

- **Development Environment:**
    
    - Backup timing isn’t critical.
        
- **Production Environment:**
    
    - Schedule backups during **low traffic** periods (because tables may be locked during backup).
        


---

## 2. Restoring Database

- Both **Simple** & **Full Recovery Models** start with restoring a **full backup**.  
- It is *important* to **test your backup** in a test database to ensure it’s correct.  
- In the **Simple Recovery Model**, no other steps are needed after restoring your last full backup.  
- In the **Full Recovery Model**, you need to restore each **transaction log** in sequence up to the specific moment you want.

### 2.1 Syntax
```sql
RESTORE DATABASE | LOG database_name
FROM DISK = ''
WITH NO RECOVERY | RECOVERY
````

- **NO RECOVERY** → database will stay in a restoring state because more backups will follow.
    
- **RECOVERY** → used when the final backup is restored (database is brought online).
    
- SQL Server uses **RECOVERY** as the default.

---

### 2.2 Restoring Full Recovery Model

#### Steps:

1. **Restore the full backup** with `NO RECOVERY` to indicate there are more backups to apply.
    
    - The database will not be available until the last transaction log is restored with `RECOVERY`.
        
2. **Restore each transaction log backup** in order until the point in time you want.
    
    - Ensures that transactions are applied in the correct sequence.
        
    - Continue using `NO RECOVERY` until the last transaction log backup.
        
    - ⚠️ You **cannot skip** a transaction log backup.

---

## 3. Modifying Tables

Changes should always be made during the **development & testing cycle** before production deployment.

---

### 3.1 Changing Columns

- Primary Key & Foreign Key columns **cannot be changed** directly.
    
- Columns with **constraints** cannot be changed unless you **remove constraints first**.

#### Syntax for altering a column:

```sql
ALTER TABLE table_name
ALTER COLUMN column_name new_datatype NULL | NOT NULL;
```

---

### 3.2 Changing Column NULLability

1️⃣ **From NOT NULL → NULL:**

- You must re-specify the column’s data type even if it doesn’t change.
    

```sql
ALTER TABLE table_name
ALTER COLUMN column_name datatype NULL;
```

2️⃣ **From NULL → NOT NULL:**

- Works **only** if there are no NULL values in the column.
    

```sql
ALTER TABLE table_name
ALTER COLUMN column_name datatype NOT NULL;
```

---

### 3.3 Changing Column Length

```sql
ALTER TABLE table_name
ALTER COLUMN column_name old_datatype(new_length) NOT NULL;
```

⚠️ **Note:** You must specify `NOT NULL`. If you don’t, SQL Server will default to `NULL`.

---

### 3.4 Changing Column Data Type

```sql
ALTER TABLE table_name
ALTER COLUMN column_name new_datatype NOT NULL;
```

---

### 3.5 Adding a New Column

- The new column will be added **at the end** of the table.
    

```sql
ALTER TABLE table_name
ADD new_column_name data_type NULL | NOT NULL DEFAULT value_of_default WITH VALUES;
```

- **WITH VALUES** → applies the default value to existing rows as well.
    
- Constraints (PK, FK, Default) can be added at the same time.
    
- ⚠️ You **cannot** add a `NOT NULL` column **without providing a DEFAULT**.
    

---

### 3.6 Dropping Columns

- Make sure the column **has no constraints**, or **drop constraints first**.
    

```sql
ALTER TABLE table_name
DROP CONSTRAINT constraint_name,
DROP COLUMN column_name;
```

- Use `sp_help table_name` in SQL Server to check constraints on the table.
    

---

### 3.7 Adding Indexes

- Choose indexes based on `WHERE` clauses you often query.
    
- Use a **prefix** like `ix_columnName` for clarity.
    
- For **small tables**, indexes might not improve performance.
    
- **Insert, update, delete** operations become slower because the index must be updated.
    

```sql
CREATE UNIQUE CLUSTERED | NONCLUSTERED INDEX index_name
ON table_name (column1);
-- You can also create a composite index:
-- ON table_name (column1, column2)
```

---

## 4. Reorganizing Data & Indexes

- We need to reorganize data when it becomes **fragmented**.  
- Multiple **INSERT**, **UPDATE**, and **DELETE** operations can shift data, which leads indexes to become **fragmented**.  
- In **SQL Server**, when you create a **Primary Key**, it is actually a **constraint**, not an index.  
  - To treat it like an index, you may need to **drop the constraint** and add a new one.

---

### 4.1 Dropping an Index

**Syntax:**
```sql
DROP INDEX table_name.index_name;
````

- When dropping and recreating an index, the RDBMS puts a **lock** on the table so no one can use it during the process.
    
- Recreating a **clustered index** is more resource-intensive than a **non-clustered index**.
    
- ⚠️ **Tip:** If you are performing a large number of **insertions**, you can:
    
    1. Drop the index.
        
    2. Load the data.
        
    3. Recreate the index after loading to avoid fragmentation.
        

---

### 4.2 DROP_EXISTING Option

**Syntax:**

```sql
CREATE UNIQUE CLUSTERED | NONCLUSTERED INDEX index_name
ON table_name (column1)
WITH (DROP_EXISTING = ON);
```

- The new index **must have the same name** as the existing index.
    
- **DROP_EXISTING** allows you to recreate the index **without manually dropping** the old one first.
    
    - ✅ This avoids placing a **lock** on the table during the process, which **improves performance**.
        

---

## 5. Building Scripts for Production Implementation

When you're ready to deploy your **development database** to the **test**, **QA**, or **production environment**, the process will depend on your company's policy.  

- Some companies may allow you to simply copy the database and clear data from certain tables.  
- Other companies insist on **SQL scripts** to recreate the entire database — this is the **more professional approach** that we’ll discuss here.

---

### 5.1 What Are Scripts?

- **Scripts** are the **SQL statements** used to create all database objects:  
  - Tables  
  - Constraints  
  - Indexes  
  - Stored Procedures  
  - Views  
  - And any other objects in the DB

How you create scripts depends on your **RDBMS** and the tools it provides.  

**Example – Microsoft SQL Server Tools:**
- **Enterprise Manager**: Can generate scripts for individual objects or for the entire DB in one file.  
- **Query Analyzer**: Allows you to create and run SQL commands, and has menu items to script out your database.  

Once generated, scripts are handed off to the **DBA** for implementation in the **test** or **production** environment.

---

### 5.2 Scripts to Populate Tables

Alongside scripts to **create objects**, you’ll often need scripts to **populate tables** that contain **static/common data** (e.g., payroll codes, country lists, etc.).  

These tables:
- Contain **required data** for your application.
- Typically **don’t change** during runtime.

✅ **Example Scenario:**  
A payroll system might have a table of **payroll types and codes** that must be populated **before** the application runs.

---

### 5.3 Why Generate Scripts Instead of Writing Manually?

- Manually writing INSERT statements is inefficient — especially for tables with many columns/rows.  
- Instead, you can **generate INSERT scripts** dynamically using SQL queries.

---

### 5.4 Example of INSERT Script Generation

Here’s the concept:  
- Use `SELECT` to **generate INSERT statements** for another environment.  
- Use **concatenation**, **string functions**, and **casting** to correctly format the output.

**SQL Example:**
```sql
SET NOCOUNT ON;

SELECT
'INSERT INTO TestTable (TableKey, SerialNum, Assigned, Notes, Price)' + 
CHAR(13) + CHAR(9) + 'VALUES(' +
CAST(TableKey AS VARCHAR(10)) + ',' +
QUOTENAME(SerialNum, '''') + ',' +
CAST(Assigned AS CHAR(1)) + ',' +
COALESCE(QUOTENAME(CAST(Notes AS VARCHAR(1000)), ''''), 'NULL') + ',' +
CAST(Price AS VARCHAR(11)) + ')'
FROM TestTable;
````

---

### 5.5 What’s Happening in the Code?

1️⃣ **SET NOCOUNT ON**

- Stops the “rows affected” message from appearing in the output.
    

2️⃣ **Building the INSERT Statement**

- Starts with a **string constant**:
    
    ```sql
    'INSERT INTO TestTable (TableKey, SerialNum, Assigned, Notes, Price)' +
    ```
    
- Adds a **carriage return** (`CHAR(13)`) and a **tab** (`CHAR(9)`) for readability.
    

3️⃣ **Casting Numeric Columns**

- SQL Server will try to **add** numbers to strings if not casted → causes errors.
    
- `CAST(TableKey AS VARCHAR(10))` safely converts numbers into strings.
    

4️⃣ **Quoting String Data**

- `QUOTENAME(SerialNum, '''')` wraps the string with single quotes.
    

5️⃣ **Handling NULLs**

- `COALESCE` ensures NULL values are handled correctly.
    
- If Notes = NULL → outputs the literal `NULL` (no quotes).
    

6️⃣ **Final Concatenation**

- Ends with `CAST(Price AS VARCHAR(11)) + ')'`.
    

---

### 5.6 Example Output

The above query will generate lines like:

```sql
INSERT INTO TestTable (TableKey, SerialNum, Assigned, Notes, Price)
VALUES(1000, '12345', 0, NULL, 0.00);

INSERT INTO TestTable (TableKey, SerialNum, Assigned, Notes, Price)
VALUES(1001, '89654', 1, 'Assigned on 28/9/2002', 0.00);

INSERT INTO TestTable (TableKey, SerialNum, Assigned, Notes, Price)
VALUES(1002, '741263985741', 1, 'Nothing new to report', 0.00);
```

---

### 5.7 Why This Method is Useful?

✅ Automates population of **lookup/static data**.  
✅ Saves **time** and avoids **manual errors**.  
✅ Works great if your DB application is **deployed for multiple customers** (each customer runs your scripts to set up and populate their DB).

---

## 6. When to Perform Database Maintenance

When performing **database maintenance tasks**, one of the most critical considerations is **timing**.  
For example, if you are running a **high-traffic e-commerce site**, you absolutely **should not** attempt to:
- Change indexes  
- Update data types  
- Reorganize tables  

… during peak usage times.

---

### 6.1 Best Practices for Choosing Maintenance Time

1️⃣ **Choose Low-Traffic Hours**  
- Perform **usage analysis** on your website or system to find out **when traffic is lowest**.  
- You can use analytics tools or even reference resources like *Practical Web Traffic Analysis* (Peter Fletcher, Glasshaus, ISBN: 1904151183).  
- The goal: **minimize downtime** and reduce disruption for users.  

> 💡 **Note:** Downtime = **lost business** for e-commerce sites and **frustration** for users on any site.

---

2️⃣ **Prepare a Maintenance Page**  
- Before starting, create a **temporary message** (e.g. “Work in Progress – Please Try Again Later”).  
- Include:
  - When the site will be back up  
  - Any helpful info to reassure users

---

3️⃣ **Perform Maintenance Work**  
- Execute your planned tasks: backup, reorganizing indexes, schema changes, etc.  
- **Test thoroughly** before pushing changes live — this prevents unexpected issues.

---

4️⃣ **Bring the Site Back Online**  
- Once verified, replace the maintenance page with your normal homepage.  
- ✅ **Pro Tip:** Provide a **feedback address** so users can report:
  - Bugs they encounter
  - Suggestions for improvement

---

### 📝 Why This Matters
- Proper scheduling ensures **minimal downtime**.  
- Users feel **informed** and **respected** when you communicate clearly.  
- Testing before going live avoids **production disasters**.

