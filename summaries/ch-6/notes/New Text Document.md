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

> [!TIP]
> To avoid confusion when joining multiple tables,or adopt prefix-based naming to avoid repetition: CustId ,CustName
> you can use table-qualified column names:


```sql
SELECT Customer.ID, Customer.Name, Product.ID, Product.Name
FROM Customer
JOIN Orders ON Customer.ID = Orders.CustomerID
JOIN OrderItem ON Orders.ID = OrderItem.OrderID
JOIN Product ON OrderItem.ProductID = Product.ID
WHERE Customer.City = 'Eureka';

```