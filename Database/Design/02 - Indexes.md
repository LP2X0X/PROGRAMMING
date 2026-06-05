---
tags: [database, design, indexes]
---

## What Is an Index?

- An **index** is a data structure maintained by the database engine that speeds up data retrieval operations on a table.
- The analogy is a book's index: instead of reading every page to find a topic, you look up the topic in the index, get a page number, and flip directly to it. A database index works the same way — instead of scanning every row in a table, the engine looks up the value in the index and jumps directly to the matching rows.
- Without an index, the database must perform a **full table scan** — reading every single row in the table to find the ones that match your query. On a table with millions of rows, this can take seconds or even minutes.
- With an appropriate index, the same query can complete in milliseconds, regardless of table size.

```ad-important
Indexes are the ==single most impactful performance tool== in relational databases. A missing index on a frequently queried column in a large table can make a query 100x to 1000x slower. Before reaching for complex optimizations, caching layers, or denormalization, always check your indexes first.
```

---

## How Indexes Work — B-Tree Structure

- The most common index structure in relational databases is the **B-tree** (balanced tree). Understanding it at a high level explains why indexes are so effective.

### Full Table Scan (No Index)

```
Query: SELECT * FROM Users WHERE Email = 'alice@email.com';

Without index — must check every row:
Row 1: bob@email.com      ← not a match
Row 2: charlie@email.com  ← not a match
Row 3: alice@email.com    ← MATCH!
Row 4: dave@email.com     ← still must check remaining rows...
...
Row 1,000,000: zara@email.com

Time complexity: O(n) — must scan all 1,000,000 rows
```

### Indexed Lookup (B-Tree)

```
Query: SELECT * FROM Users WHERE Email = 'alice@email.com';

With B-tree index on Email — tree traversal:

                     [M]
                    /   \
               [D]       [S]
              /   \     /   \
           [A-C] [E-L] [N-R] [T-Z]
            |
         alice@email.com → Row pointer → Row 3

Time complexity: O(log n) — finds row in ~20 steps for 1,000,000 rows
```

- The B-tree keeps data sorted and balanced. At each level of the tree, the engine makes a single comparison to decide which branch to follow. With a million rows, the tree is roughly 20 levels deep — so the engine makes about 20 comparisons instead of 1,000,000.
- This is the difference between a query that takes 500ms and one that takes 0.5ms.

```ad-note
B-tree is not the only index type. Other types include **hash indexes** (exact match only, no range queries), **GiST/GIN indexes** (for full-text search and spatial data in PostgreSQL), and **bitmap indexes** (for low-cardinality columns in data warehouses). But B-tree is the default and most broadly useful — when someone says "index" without qualification, they mean a B-tree index.
```

---

## Clustered vs Non-Clustered Indexes

- This is one of the most important distinctions in index architecture. Every table can have at most **one clustered index** and **many non-clustered indexes**.

### Clustered Index

- A **clustered index** determines the **physical order** of the data rows on disk. The table data *is* the index — the leaf nodes of the B-tree contain the actual data rows.
- Because data can only be physically sorted in one order, a table can have **only one** clustered index.
- By default, the [[Primary Key|primary key]] creates a clustered index (in SQL Server and MySQL/InnoDB). This means the table's rows are physically ordered by the primary key.

**Analogy:** Think of an encyclopedia. The articles are physically sorted A-Z — the book itself *is* the index. You don't need a separate lookup structure to find "Algorithm" — you open near the front.

### Non-Clustered Index

- A **non-clustered index** is a **separate data structure** from the table data. The leaf nodes of the B-tree contain the indexed column values and a **pointer** (row locator) back to the actual data row in the table.
- A table can have **many** non-clustered indexes — each one provides a different "lookup path" into the same data.

**Analogy:** Think of a book's index at the back. It lists topics alphabetically with page numbers. The index is separate from the content — you look up the topic, get the page number, then flip to that page to read the actual content.

### Comparison

| | Clustered Index | Non-Clustered Index |
| --- | --- | --- |
| **What it is** | The table data itself, sorted by the index key | A separate structure pointing to the table data |
| **How many per table?** | Exactly ONE | Many (practical limit varies; typically 5-15 is reasonable) |
| **Leaf nodes contain** | The actual data rows | Index key + pointer to the data row |
| **Created by default** | Primary key (in SQL Server, MySQL/InnoDB) | Must be created manually |
| **Speed for key lookups** | Fastest — data is right there | Slightly slower — must follow the pointer to the data row |
| **Speed for range scans** | Excellent — data is physically contiguous | Good, but may require many pointer lookups |
| **Analogy** | Encyclopedia sorted A-Z | Book index at the back |

### How a Non-Clustered Index Lookup Works

```
1. Query: WHERE Email = 'alice@email.com'
2. Engine searches the non-clustered index B-tree for 'alice@email.com'
3. Finds the entry: 'alice@email.com' → Row pointer (page 42, slot 7)
4. Follows the pointer to the clustered index (the actual table)
5. Reads the full row from page 42, slot 7

This extra step (#4) is called a "key lookup" or "bookmark lookup"
```

```ad-note
The key lookup is why non-clustered indexes are slightly slower than clustered indexes for single-row access — there is an extra hop. For queries returning many rows, this overhead multiplies. This is where **covering indexes** (discussed below) become valuable — they eliminate the key lookup entirely.
```

---

## Creating Indexes

### Basic Syntax

```sql
-- Single-column index
CREATE INDEX IX_Users_Email ON Users(Email);

-- Composite index (multiple columns)
CREATE INDEX IX_Orders_CustomerId_OrderDate ON Orders(CustomerId, OrderDate);

-- Unique index — also enforces uniqueness (like a UNIQUE constraint)
CREATE UNIQUE INDEX IX_Users_Email ON Users(Email);

-- Drop an index
DROP INDEX IX_Users_Email ON Users;
```

### Composite Index Column Order Matters

- In a composite index, the **column order determines which queries the index can serve**.
- A composite index on `(CustomerId, OrderDate)` can efficiently serve:
  - `WHERE CustomerId = 5` (uses the first column)
  - `WHERE CustomerId = 5 AND OrderDate > '2024-01-01'` (uses both columns)
- But it **cannot** efficiently serve:
  - `WHERE OrderDate > '2024-01-01'` alone (the first column is skipped — the index cannot be used)

```ad-warning
This is the **leftmost prefix rule** — a composite index can only be used if the query filters on a leftmost prefix of the index columns. Think of it like a phone book sorted by (LastName, FirstName): you can look up everyone named "Smith", or "Smith, John", but you cannot efficiently look up everyone named "John" regardless of last name — the book is not sorted that way.
```

- **Rule of thumb for column order in composite indexes:**
  1. Columns used in **equality conditions** (`=`) first
  2. Then columns used in **range conditions** (`>`, `<`, `BETWEEN`) or **ORDER BY**

```sql
-- Query pattern:
SELECT * FROM Orders
WHERE CustomerId = 5 AND Status = 'Shipped'
ORDER BY OrderDate DESC;

-- Best composite index for this query:
CREATE INDEX IX_Orders_CustId_Status_Date
ON Orders(CustomerId, Status, OrderDate);
-- Equality columns first (CustomerId, Status), then range/sort column (OrderDate)
```

---

## When to Create an Index

- **Columns in `WHERE` clauses** that are queried frequently — the most common reason to index.
- **Columns in `JOIN` conditions** — [[Foreign Key|foreign key]] columns should almost always be indexed. The database needs to find matching rows in the joined table quickly.
- **Columns in `ORDER BY` clauses** — an index can provide pre-sorted data, avoiding an expensive sort operation.
- **Columns in `GROUP BY` clauses** — similar to `ORDER BY`, grouping benefits from sorted data.
- **Columns with high selectivity** — columns where the indexed value narrows the result set significantly (e.g., `Email`, `OrderId`, `SSN`).

```sql
-- These queries strongly suggest needing indexes:

-- 1. WHERE clause
SELECT * FROM Users WHERE Email = 'alice@email.com';
-- → CREATE INDEX IX_Users_Email ON Users(Email);

-- 2. JOIN condition
SELECT o.*, c.Name
FROM Orders o
JOIN Customers c ON o.CustomerId = c.Id;
-- → CREATE INDEX IX_Orders_CustomerId ON Orders(CustomerId);

-- 3. ORDER BY
SELECT * FROM Products ORDER BY Price DESC;
-- → CREATE INDEX IX_Products_Price ON Products(Price);

-- 4. Combination
SELECT * FROM Orders
WHERE CustomerId = 5
ORDER BY OrderDate DESC;
-- → CREATE INDEX IX_Orders_CustId_Date ON Orders(CustomerId, OrderDate);
```

---

## When NOT to Create an Index

- **Small tables** — if the table has fewer than a few hundred rows, a full table scan is actually faster than an index lookup. The overhead of maintaining the index outweighs the benefit.
- **Columns that change very frequently** — every `INSERT`, `UPDATE`, or `DELETE` must also update the index. If a column is updated thousands of times per second, the index maintenance cost may outweigh the read benefit.
- **Low-cardinality columns** — columns with very few distinct values (e.g., `Gender` with M/F/Other, `IsActive` with 0/1). An index on such a column does not narrow the result set much — the database may still need to read half the table.
- **Columns rarely used in queries** — if a column is never used in `WHERE`, `JOIN`, `ORDER BY`, or `GROUP BY`, an index on it is pure overhead.
- **Tables with heavy write workloads and rare reads** — log tables, event streams, and audit trails that are written constantly but queried rarely may not benefit from many indexes.

```ad-note
The exception to the low-cardinality rule: if you are querying for the *rare* value in a low-cardinality column, an index can help. For example, if 99.9% of rows have `IsActive = 1` and you often query `WHERE IsActive = 0`, an index can quickly find the tiny number of inactive rows. This is sometimes called a "selective index" scenario. Some databases support **filtered indexes** (SQL Server) or **partial indexes** (PostgreSQL) specifically for this use case.
```

---

## The Cost of Indexes

- Indexes are not free. They provide faster reads at the expense of other resources:

| Cost | Explanation |
| --- | --- |
| **Slower writes** | Every `INSERT` must add an entry to every index on the table. Every `UPDATE` to an indexed column must update the index. Every `DELETE` must remove the index entry. |
| **Disk space** | Each index is a separate data structure stored on disk. A non-clustered index on a large table can be hundreds of MB. |
| **Memory usage** | Frequently accessed index pages are cached in the buffer pool, consuming RAM that could be used for data pages. |
| **Maintenance overhead** | Indexes can become **fragmented** over time as rows are inserted, updated, and deleted. Periodic index rebuilds or reorganizations are needed. |
| **Optimizer confusion** | Too many indexes can cause the query optimizer to choose suboptimal execution plans — it has too many options to evaluate. |

```ad-warning
A common anti-pattern is "index everything." New developers sometimes create an index on every column in every table. This dramatically slows down write operations (every insert now updates dozens of indexes) and wastes disk space and memory. Be deliberate — index the columns that your actual queries need.
```

### Finding the Balance

- The general guideline: **start with indexes on primary keys (automatic), foreign keys, and the most frequently queried columns**. Monitor query performance, and add indexes when you identify slow queries that would benefit from them.
- Most OLTP (Online Transaction Processing) tables perform well with **3-8 indexes**. Reporting/analytical tables may have more.

---

## Covering Indexes

- A **covering index** is a non-clustered index that contains *all* the columns a query needs — so the database can satisfy the query entirely from the index without ever looking up the actual table row.

### The Problem: Key Lookups

```sql
-- Query:
SELECT Name, Email FROM Users WHERE Email = 'alice@email.com';

-- Index: IX_Users_Email ON Users(Email)
-- The index contains Email but NOT Name
-- So the engine:
-- 1. Finds 'alice@email.com' in the index → gets row pointer
-- 2. Follows the pointer to the table (key lookup) → reads the full row
-- 3. Extracts Name from the full row
```

- The key lookup in step 2 is an extra I/O operation. For a single row it is negligible, but for a query returning thousands of rows, it means thousands of extra random I/O operations.

### The Solution: Include the Extra Columns in the Index

```sql
-- Covering index: Email is the search key, Name is included for coverage
CREATE INDEX IX_Users_Email_Name ON Users(Email) INCLUDE (Name);

-- Now the engine:
-- 1. Finds 'alice@email.com' in the index
-- 2. Name is RIGHT THERE in the index leaf node — no key lookup needed!
-- 3. Returns the result directly from the index
```

- The `INCLUDE` keyword adds columns to the **leaf level** of the index without making them part of the index key. This means:
  - The column is available for reading without a key lookup
  - But the column is **not** used for sorting or searching within the index
  - The column does **not** contribute to the index key size

```sql
-- More examples:

-- Query: SELECT OrderId, OrderDate, TotalAmount
--        FROM Orders WHERE CustomerId = 5 ORDER BY OrderDate;
CREATE INDEX IX_Orders_CustId_Date
ON Orders(CustomerId, OrderDate)
INCLUDE (TotalAmount);
-- CustomerId and OrderDate are key columns (used for search and sort)
-- TotalAmount is included (just needs to be available in the result)

-- Query: SELECT ProductName, Price FROM Products WHERE Category = 'Electronics';
CREATE INDEX IX_Products_Category
ON Products(Category)
INCLUDE (ProductName, Price);
```

```ad-note
In SQL Server, you can check whether a query is doing key lookups by examining the **execution plan**. Look for "Key Lookup" or "RID Lookup" operators — these indicate that the index does not cover all the columns the query needs, and a covering index might help.
```

---

## Index Types Summary

| Index Type | Description | Use Case |
| --- | --- | --- |
| **Clustered** | Table data sorted by index key; one per table | Primary key lookups, range scans on the PK |
| **Non-clustered** | Separate structure pointing to table rows; many per table | Lookups on non-PK columns |
| **Unique** | Non-clustered + enforces uniqueness | Columns that must be unique (e.g., Email) |
| **Composite** | Index on multiple columns | Queries filtering on multiple columns |
| **Covering** | Non-clustered with INCLUDE columns | Eliminating key lookups for specific queries |
| **Filtered** (SQL Server) | Index with a WHERE clause — only indexes qualifying rows | Indexing rare values in a column (e.g., `WHERE IsActive = 0`) |

```sql
-- Filtered index example (SQL Server):
CREATE INDEX IX_Orders_Pending
ON Orders(OrderDate)
WHERE Status = 'Pending';
-- Only indexes the small number of pending orders — much smaller and faster
```

---

## Practical Guidelines — Index Decision Checklist

When deciding whether to add an index, ask these questions:

1. **Is this column used in `WHERE`, `JOIN`, `ORDER BY`, or `GROUP BY` in a frequently executed query?** If no, don't index it.
2. **Is the table large enough that a full scan is noticeable?** If the table has fewer than a few hundred rows, an index won't help.
3. **Does the column have good selectivity?** If querying the column narrows the result set to a small percentage of rows, an index helps. If it still returns half the table, it might not.
4. **Is the table write-heavy?** Every index slows writes. If the table receives thousands of inserts per second (e.g., a log table), be very selective about indexes.
5. **Is there an existing composite index that already covers this column?** Don't create a redundant index.

```ad-important
The most common real-world index mistake is not having an index on **foreign key columns**. When you `JOIN Orders ON Customers.Id = Orders.CustomerId`, the database needs to look up matching rows in `Orders` by `CustomerId`. If `CustomerId` is not indexed, this is a full table scan on `Orders` for *every row* in `Customers`. Always index your foreign keys.
```

---

## Viewing and Managing Indexes

```sql
-- SQL Server: list all indexes on a table
EXEC sp_helpindex 'Users';

-- SQL Server: show index usage statistics
SELECT * FROM sys.dm_db_index_usage_stats
WHERE object_id = OBJECT_ID('Users');

-- MySQL/MariaDB: list indexes on a table
SHOW INDEX FROM Users;

-- Rebuild a fragmented index (SQL Server)
ALTER INDEX IX_Users_Email ON Users REBUILD;

-- Reorganize a fragmented index (lighter operation)
ALTER INDEX IX_Users_Email ON Users REORGANIZE;
```

```ad-note
Index fragmentation occurs when the logical order of index pages diverges from the physical order on disk, usually due to many inserts and deletes. Fragmented indexes cause extra I/O. As a general rule: **reorganize** when fragmentation is 10-30%, **rebuild** when it exceeds 30%. Most production databases schedule index maintenance jobs to run during off-peak hours.
```

---

**Next:** [[03 - Constraints]]
