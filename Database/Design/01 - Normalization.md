---
tags: [database, design, normalization]
---

## What Is Normalization?

- **Normalization** is the process of organizing the columns and tables in a relational database to reduce data redundancy and prevent data anomalies.
- The core idea: *each piece of information should be stored in exactly one place*. When a fact is stored in multiple rows or tables, it creates opportunities for the copies to get out of sync — and once they do, you no longer know which copy is correct.
- Normalization was formalized by **Edgar F. Codd** in 1970 as part of the relational model. It provides a set of progressive rules called **normal forms** (1NF, 2NF, 3NF, etc.) that each eliminate a specific category of redundancy.
- In practice, most production databases are normalized to **Third Normal Form (3NF)** — this eliminates the vast majority of redundancy issues while keeping the schema practical to work with. Higher normal forms (BCNF, 4NF, 5NF) exist but are rarely needed in day-to-day application development.

---

## Why Normalize? — Data Anomalies

- Without normalization, redundant data leads to three categories of problems called **data anomalies**. These are not theoretical — they are bugs waiting to happen in any unnormalized schema.

### The Unnormalized Example

- Consider this single table that stores order, customer, and product data all in one place:

```sql
-- BAD: Unnormalized Orders table
-- Everything crammed into a single table
CREATE TABLE Orders (
    OrderId     INT,
    CustomerName VARCHAR(100),
    CustomerEmail VARCHAR(100),
    Product      VARCHAR(100),
    ProductPrice DECIMAL(10,2),
    Qty          INT
);
```

| OrderId | CustomerName | CustomerEmail  | Product | ProductPrice | Qty |
| ------- | ------------ | -------------- | ------- | ------------ | --- |
| 1       | Long         | long@email.com | Widget  | 9.99         | 2   |
| 2       | Long         | long@email.com | Gadget  | 19.99        | 1   |
| 3       | Alice        | alice@email.com| Widget  | 9.99         | 3   |

- This table *works* — you can query it, insert into it, and get results. But it has serious structural problems that will surface as the data grows.

### Update Anomaly

- Long's email (`long@email.com`) is stored in **two rows** (OrderId 1 and 2). If Long changes his email address, you must update *every row* that mentions him.
- If you update row 1 but forget row 2, Long now has two different email addresses in the database. Which one is correct? Neither the application nor the database can tell.

```sql
-- Oops — only updated one row
UPDATE Orders SET CustomerEmail = 'long.new@email.com' WHERE OrderId = 1;

-- Now the data is inconsistent:
-- OrderId 1: long.new@email.com
-- OrderId 2: long@email.com         ← stale!
```

- In a normalized design, Long's email is stored in *one row* in a Customers table. One `UPDATE`, guaranteed consistency.

### Insert Anomaly

- Suppose you sign up a new customer, Bob, but he has not placed any orders yet. Can you insert Bob into this table?
- **No** — because `OrderId`, `Product`, `ProductPrice`, and `Qty` are required parts of the row. You would have to either leave them `NULL` (if the schema allows it, which is semantically wrong — what is a "null order"?) or invent a fake order just to store Bob's information.
- In a normalized design, Bob goes into the Customers table regardless of whether he has orders.

### Delete Anomaly

- Alice (OrderId 3) has only one order. If she cancels that order and you delete the row, you lose *all* information about Alice — her name, her email, everything — because it was only stored in that one order row.
- In a normalized design, deleting an order only removes the order. Alice's customer record remains intact in the Customers table.

```ad-warning
Data anomalies are not edge cases that "probably won't happen." In production databases with thousands of rows and multiple application writers, they *will* happen. Normalization is the structural solution — it makes entire categories of bugs impossible by design, not by discipline.
```

---

## First Normal Form (1NF)

- **1NF** establishes the most basic requirements for a relational table. A table is in 1NF if:

1. **Each cell contains a single, atomic value** — no lists, no comma-separated values, no repeated groups within a single column.
2. **Each row is uniquely identifiable** — the table has a [[Primary Key|primary key]].
3. **Each column has a consistent data type** — every value in a column is of the same type.
4. **The order of rows and columns does not matter** — the data is not dependent on physical arrangement.

### Violating 1NF — Multi-valued Columns

```sql
-- BAD: PhoneNumbers column stores multiple values in one cell
CREATE TABLE Contacts (
    ContactId    INT PRIMARY KEY,
    Name         VARCHAR(100),
    PhoneNumbers VARCHAR(500)  -- stores "555-1234, 555-5678, 555-9999"
);
```

| ContactId | Name  | PhoneNumbers                |
| --------- | ----- | --------------------------- |
| 1         | Long  | 555-1234, 555-5678          |
| 2         | Alice | 555-9999                    |
| 3         | Bob   | 555-1111, 555-2222, 555-3333|

- **Problems with this design:**
  - How do you query "find the contact with phone number 555-5678"? You need `LIKE '%555-5678%'` — which is slow (no index can help), fragile (what about "555-56789"?), and ugly.
  - How do you delete one phone number from a contact? You have to parse the string, remove the number, and rewrite the entire cell.
  - How do you count how many phone numbers each contact has? String parsing again.
  - What is the maximum number of phone numbers? The `VARCHAR(500)` limit silently caps it.

### Fixing the 1NF Violation

```sql
-- GOOD: Separate table for phone numbers
CREATE TABLE Contacts (
    ContactId INT PRIMARY KEY,
    Name      VARCHAR(100)
);

CREATE TABLE PhoneNumbers (
    PhoneId   INT PRIMARY KEY,
    ContactId INT REFERENCES Contacts(ContactId),
    Phone     VARCHAR(20)
);
```

| PhoneId | ContactId | Phone    |
| ------- | --------- | -------- |
| 1       | 1         | 555-1234 |
| 2       | 1         | 555-5678 |
| 3       | 2         | 555-9999 |
| 4       | 3         | 555-1111 |
| 5       | 3         | 555-2222 |
| 6       | 3         | 555-3333 |

- Now each phone number is its own row. Querying, indexing, inserting, and deleting individual phone numbers is trivial.

```ad-note
1NF violations are extremely common in real-world databases, especially when developers try to avoid creating "too many tables." Comma-separated values in a column are the biggest red flag — if you see this pattern, it almost always needs to be refactored into a separate table with a [[Foreign Key|foreign key]] relationship.
```

### Violating 1NF — Repeating Groups

- Another form of 1NF violation is **repeating groups** — multiple columns that represent the same kind of data:

```sql
-- BAD: Repeating groups — what if a student takes a 4th course?
CREATE TABLE StudentCourses (
    StudentId INT PRIMARY KEY,
    Course1   VARCHAR(100),
    Course2   VARCHAR(100),
    Course3   VARCHAR(100)
);
```

- This caps students at 3 courses, wastes space when students take fewer, and makes querying ("find all students taking Database 101") require checking three columns. The fix is the same: a separate enrollment table.

---

## Second Normal Form (2NF)

- **2NF** builds on 1NF. A table is in 2NF if:

1. It is already in **1NF**.
2. **Every non-key column depends on the *entire* primary key**, not just part of it.

- 2NF violations **only occur in tables with composite (multi-column) primary keys**. If your table has a single-column primary key, it is automatically in 2NF (assuming it is already in 1NF).

### What Is a Partial Dependency?

- A **partial dependency** occurs when a non-key column depends on *part* of a composite primary key, not the full key.

```sql
-- BAD: Partial dependency
-- Primary key is (StudentId, CourseId)
-- But StudentName depends ONLY on StudentId — not on CourseId
CREATE TABLE Enrollments (
    StudentId   INT,
    CourseId     INT,
    StudentName VARCHAR(100),   -- depends only on StudentId (partial dependency!)
    CourseName  VARCHAR(100),   -- depends only on CourseId (partial dependency!)
    Grade       CHAR(1),        -- depends on (StudentId, CourseId) — this is fine
    PRIMARY KEY (StudentId, CourseId)
);
```

| StudentId | CourseId | StudentName | CourseName    | Grade |
| --------- | ------- | ----------- | ------------- | ----- |
| 1         | 101     | Long        | Database 101  | A     |
| 1         | 102     | Long        | Calculus      | B     |
| 2         | 101     | Alice       | Database 101  | A     |

- `StudentName` is the same regardless of which course — it depends only on `StudentId`. This is a partial dependency.
- `CourseName` is the same regardless of which student — it depends only on `CourseId`. Another partial dependency.
- `Grade` depends on the *combination* of student and course — this is a full dependency and is fine.

- **The problem:** "Long" appears in two rows, "Database 101" appears in two rows. Update anomalies again.

### Fixing the 2NF Violation

- Move partially dependent columns into their own tables:

```sql
-- GOOD: Each fact stored once
CREATE TABLE Students (
    StudentId   INT PRIMARY KEY,
    StudentName VARCHAR(100)
);

CREATE TABLE Courses (
    CourseId    INT PRIMARY KEY,
    CourseName  VARCHAR(100)
);

CREATE TABLE Enrollments (
    StudentId INT REFERENCES Students(StudentId),
    CourseId  INT REFERENCES Courses(CourseId),
    Grade     CHAR(1),
    PRIMARY KEY (StudentId, CourseId)
);
```

- Now `StudentName` is stored once per student, `CourseName` is stored once per course, and `Enrollments` contains only the data that genuinely depends on the combination of student and course.

```ad-note
In practice, 2NF violations are less common than 1NF violations because most tables use a single-column surrogate key (`Id INT PRIMARY KEY IDENTITY`). But when you do use composite keys — particularly in junction/bridge tables — always check for partial dependencies.
```

---

## Third Normal Form (3NF)

- **3NF** builds on 2NF. A table is in 3NF if:

1. It is already in **2NF**.
2. **No non-key column depends on another non-key column** — i.e., no **transitive dependencies**.

### What Is a Transitive Dependency?

- A **transitive dependency** occurs when a non-key column determines another non-key column, creating a chain:
  - PK --> Column A --> Column B
  - Column B depends on Column A, which depends on the PK. Column B is *transitively* dependent on the PK.

```sql
-- BAD: Transitive dependency
-- PK is EmployeeId
-- DepartmentName depends on DepartmentId (not on EmployeeId directly)
CREATE TABLE Employees (
    EmployeeId     INT PRIMARY KEY,
    EmployeeName   VARCHAR(100),
    DepartmentId   INT,
    DepartmentName VARCHAR(100)   -- depends on DepartmentId, not EmployeeId!
);
```

| EmployeeId | EmployeeName | DepartmentId | DepartmentName |
| ---------- | ------------ | ------------ | -------------- |
| 1          | Long         | 10           | Engineering    |
| 2          | Alice        | 10           | Engineering    |
| 3          | Bob          | 20           | Marketing      |

- The chain: `EmployeeId` --> `DepartmentId` --> `DepartmentName`.
- `DepartmentName` is stored once per employee, not once per department. "Engineering" appears twice. If the department is renamed, you must update every employee row in that department.

### Fixing the 3NF Violation

```sql
-- GOOD: Separate Departments table breaks the transitive dependency
CREATE TABLE Departments (
    DepartmentId   INT PRIMARY KEY,
    DepartmentName VARCHAR(100)
);

CREATE TABLE Employees (
    EmployeeId   INT PRIMARY KEY,
    EmployeeName VARCHAR(100),
    DepartmentId INT REFERENCES Departments(DepartmentId)
);
```

- Now `DepartmentName` is stored exactly once, in the Departments table. The Employees table references it via `DepartmentId` ([[Foreign Key|foreign key]]).

### Another Classic Example: ZipCode --> City, State

```sql
-- BAD: City and State depend on ZipCode, not on the PK
CREATE TABLE Addresses (
    AddressId INT PRIMARY KEY,
    Street    VARCHAR(200),
    City      VARCHAR(100),
    State     VARCHAR(50),
    ZipCode   VARCHAR(10)
);
```

- `ZipCode` --> `City` and `ZipCode` --> `State`. If ZipCode 90210 appears in 500 rows, "Beverly Hills" and "California" are stored 500 times.

```sql
-- GOOD: Separate ZipCodes table
CREATE TABLE ZipCodes (
    ZipCode VARCHAR(10) PRIMARY KEY,
    City    VARCHAR(100),
    State   VARCHAR(50)
);

CREATE TABLE Addresses (
    AddressId INT PRIMARY KEY,
    Street    VARCHAR(200),
    ZipCode   VARCHAR(10) REFERENCES ZipCodes(ZipCode)
);
```

```ad-note
The ZipCode example is a classic textbook case, but in practice many teams choose to *not* normalize this far for addresses — because zip code data is messy (one zip can span multiple cities, cities change), and address data is often treated as a flat blob from a third-party validation service. This is an example where practical considerations override pure normalization theory. See the [[#Denormalization]] section below.
```

---

## The Normalization Process — Step by Step

- Let's normalize the original bad `Orders` table through all three normal forms.

### Starting Point: Unnormalized

| OrderId | CustomerName | CustomerEmail   | Product | ProductPrice | Qty |
| ------- | ------------ | --------------- | ------- | ------------ | --- |
| 1       | Long         | long@email.com  | Widget  | 9.99         | 2   |
| 2       | Long         | long@email.com  | Gadget  | 19.99        | 1   |
| 3       | Alice        | alice@email.com | Widget  | 9.99         | 3   |

### Step 1: Apply 1NF

- The table is already in 1NF — each cell has one value, there are no repeating groups, and `OrderId` serves as a primary key.

### Step 2: Apply 2NF

- The primary key is a single column (`OrderId`), so there are no partial dependencies. The table is already in 2NF.

### Step 3: Apply 3NF — Remove Transitive Dependencies

- Identify the transitive dependencies:
  - `OrderId` --> `CustomerName` and `OrderId` --> `CustomerEmail`. But `CustomerEmail` depends on the *customer*, not the order. If the same customer has multiple orders, both `CustomerName` and `CustomerEmail` are repeated.
  - `OrderId` --> `Product` and `OrderId` --> `ProductPrice`. But `ProductPrice` depends on the *product*, not the order. If the same product appears in multiple orders, both `Product` and `ProductPrice` are repeated.

- Solution: extract Customers and Products into their own tables:

```sql
-- Customers: each customer stored once
CREATE TABLE Customers (
    CustomerId    INT PRIMARY KEY,
    CustomerName  VARCHAR(100),
    CustomerEmail VARCHAR(100)
);

-- Products: each product stored once
CREATE TABLE Products (
    ProductId    INT PRIMARY KEY,
    ProductName  VARCHAR(100),
    ProductPrice DECIMAL(10,2)
);

-- Orders: only stores the relationships and order-specific data
CREATE TABLE Orders (
    OrderId    INT PRIMARY KEY,
    CustomerId INT REFERENCES Customers(CustomerId),
    ProductId  INT REFERENCES Products(ProductId),
    Qty        INT
);
```

### The Resulting Data

**Customers:**

| CustomerId | CustomerName | CustomerEmail   |
| ---------- | ------------ | --------------- |
| 1          | Long         | long@email.com  |
| 2          | Alice        | alice@email.com |

**Products:**

| ProductId | ProductName | ProductPrice |
| --------- | ----------- | ------------ |
| 1         | Widget      | 9.99         |
| 2         | Gadget      | 19.99        |

**Orders:**

| OrderId | CustomerId | ProductId | Qty |
| ------- | ---------- | --------- | --- |
| 1       | 1          | 1         | 2   |
| 2       | 1          | 2         | 1   |
| 3       | 2          | 1         | 3   |

- Long's email is stored **once**. Widget's price is stored **once**. All three anomaly types are eliminated.
- To get the full order details, you use a [[JOIN]]:

```sql
SELECT o.OrderId, c.CustomerName, c.CustomerEmail,
       p.ProductName, p.ProductPrice, o.Qty,
       p.ProductPrice * o.Qty AS TotalPrice
FROM Orders o
JOIN Customers c ON o.CustomerId = c.CustomerId
JOIN Products p ON o.ProductId = p.ProductId;
```

---

## Beyond 3NF — Higher Normal Forms

- For completeness, here is a brief overview of higher normal forms. In practice, reaching 3NF is sufficient for the vast majority of database designs.

| Normal Form | Rule | When It Matters |
| ----------- | ---- | --------------- |
| **BCNF** (Boyce-Codd NF) | Every determinant is a candidate key | When a table has multiple overlapping candidate keys |
| **4NF** | No multi-valued dependencies | When two independent multi-valued facts are stored in one table |
| **5NF** | No join dependencies | Extremely rare — decomposition of complex relationships |

```ad-note
If you are designing application databases (not academic exercises), 3NF is your target. BCNF occasionally matters when dealing with complex candidate key situations. 4NF and 5NF are almost exclusively discussed in database theory courses and certification exams.
```

---

## Denormalization

- **Denormalization** is the intentional process of *re-introducing* redundancy into a normalized schema to improve read performance.
- This is *not* the same as having a poorly designed schema — denormalization is a conscious, measured trade-off made *after* normalizing first.

### When to Denormalize

- **Reporting and analytics** — a reporting database that runs complex aggregation queries may benefit from pre-joined, pre-aggregated tables to avoid expensive multi-table joins at query time.
- **Read-heavy workloads** — if a query joining 5 tables runs thousands of times per second, materializing the result into a single table (or a [[materialized view]]) can dramatically reduce query time.
- **Caching frequently accessed computed values** — storing `TotalOrderAmount` directly on the `Orders` table instead of computing `SUM(LineItems.Price * LineItems.Qty)` every time.

### When NOT to Denormalize

- "My queries are slow" — the answer is usually a missing [[02 - Indexes|index]], not denormalization.
- "JOINs are slow" — they are not, when tables are properly indexed. JOINs on indexed foreign keys are extremely fast.
- "It's simpler without all these tables" — simplicity in schema leads to complexity in maintenance. A flat table is simple to read but painful to keep consistent.

```ad-important
"Normalize until it hurts, denormalize until it works." This is the professional approach — start with a properly normalized schema, measure actual performance under realistic load, and only then selectively denormalize the specific areas where you have evidence of a bottleneck. Never start denormalized.
```

### Denormalization Example

```sql
-- Normalized: must JOIN to get the total
SELECT o.OrderId, SUM(li.Price * li.Qty) AS Total
FROM Orders o
JOIN LineItems li ON o.OrderId = li.OrderId
GROUP BY o.OrderId;

-- Denormalized: total stored directly on the order (redundant, but fast)
-- You must now keep this in sync whenever line items change!
ALTER TABLE Orders ADD TotalAmount DECIMAL(10,2);
```

- The trade-off: reads are faster, but every `INSERT`, `UPDATE`, or `DELETE` on `LineItems` must also update `Orders.TotalAmount`. If your application forgets to do this, the data becomes inconsistent. This is exactly the kind of anomaly normalization was designed to prevent — you are choosing to accept this risk in exchange for read performance.

---

## Quick Reference — Normal Forms Summary

| Normal Form | Key Rule | Fix |
| ----------- | -------- | --- |
| **1NF** | Atomic values, no repeating groups | Move multi-valued data to a separate table |
| **2NF** | No partial dependencies (full key dependence) | Move partially dependent columns to their own table |
| **3NF** | No transitive dependencies | Move transitively dependent columns to their own table |

```ad-note
A helpful mnemonic for 3NF: "Every non-key column must provide a fact about *the key* (1NF), *the whole key* (2NF), and *nothing but the key* (3NF) — so help me Codd."
```

---

**Next:** [[02 - Indexes]]
