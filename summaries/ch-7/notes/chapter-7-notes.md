# 🔐 Database Security

This section highlights best practices and strategies for protecting your database and its data during development and deployment.

---

## 1️⃣ Designing a Security Model

> Database security must be planned as part of the development process, not as an afterthought.

### ✅ Goals:
- Protect sensitive data from unauthorized access or modification.
- Enforce only required levels of access to users and systems.

### 🔄 Key Components of a Security Model:
1. **Target Audience** – Who will access the data?
2. **Authentication Method** – How will they log in?
3. **Access Level** – What can each user or role do (read/insert/update/delete)?



## 👥 Identifying Your Target Audience

> Understanding your users helps tailor security effectively.

| User Type     | Typical Access Needed                             |
|---------------|----------------------------------------------------|
| Customer      | Mostly **read-only** (e.g., view products)         |
| Admin Staff   | **Full access** to modify product/inventory data   |
| External App  | Varies depending on integration role               |

✅ Tips:
- Analyze application workflows to determine access needs.
- Don’t forget **admins**, **automated jobs**, or **backend services**.
- Only grant permissions necessary for each user's job.



## 🔑 Choosing an Authentication Method

> Authentication defines **how** users are verified when accessing the database.

### 🔐 Common Authentication Types:

| Method                   | Description                                                                 |
|--------------------------|-----------------------------------------------------------------------------|
| **OS Authentication**    | Uses OS-level user accounts (e.g., Windows domain). Ideal for internal use. |
| **Database Authentication** | Usernames/passwords stored in the DBMS itself. Ideal for external users.   |

### 💡 Example: SQL Server Modes

| Mode                     | Use Case                                            |
|--------------------------|-----------------------------------------------------|
| Windows Authentication   | Internal staff, developers, DBAs                    |
| SQL Server Authentication| Public website users, APIs, partner integrations    |
| Mixed Mode               | Best practice for web-connected SQL Server setups   |

> 🔒 Use **Mixed Mode Authentication** in SQL Server for flexibility and separation of internal and external access.


### 🛡️ Best Practices Summary

- ✅ Always **identify your users** and their access levels early in development.
- ✅ Choose an authentication method that fits your system and audience.
- ✅ Prefer **Windows Authentication** for internal use.
- ✅ Use **SQL Authentication** (or database-defined users) for web apps or external services.
- ⚠️ Avoid exposing system accounts or OS-level credentials to public applications.


## 🔍 Identifying the Type of Data Access Needed

Once your database schema is defined, it's important to control how data is accessed based on the application's needs and security requirements.



### 🎯 Understand the Data Access Scenarios

In most applications (e.g., e-commerce), different parts of the user flow require different types of access:

| Scenario                     | Access Type        | Purpose                                                             |
|------------------------------|--------------------|----------------------------------------------------------------------|
| Product search, browsing     | **Read-only**       | Display product details, prices, reviews                            |
| Customer registration        | **Insert/Update**   | Save user details like shipping address, payment info               |
| Admin dashboard              | **Insert/Update/Delete** | Modify inventory, orders, customer accounts                    |

> 🧠 Most traffic involves **reading** data. Write operations are typically limited to specific workflows like checkout or admin actions.



### 🛡️ Best Practice: Use Two Separate Logins

Using **separate database logins** based on access needs helps reduce risk.

| Login Type       | Permissions           | Use Case                                 |
|------------------|-----------------------|-------------------------------------------|
| `read_user`      | `SELECT` only         | Public website features (e.g., browsing)  |
| `write_user`     | `INSERT`, `UPDATE`    | Secure pages like checkout, account info  |

> ⚠️ Avoid using a single login with full privileges throughout the application.



### 🧩 Why Two Logins?

- ✅ **Limits exposure** if credentials are leaked or exploited.
- ✅ **Prevents unnecessary write access** from public-facing pages.
- ✅ **Enhances control** over what parts of the site perform write operations.
- 🚫 Using one login with all permissions increases the **attack surface**.



### 🔧 Implementation Tip

Review your application workflows and clearly separate:
- Pages that only **read** data (e.g., product listing, search).
- Pages that **write** data (e.g., checkout, account settings).

Use the appropriate login credentials depending on the function:

---