---
tags: [performance, optimization, queries]
---

- A slow query is rarely a mystery. The vast majority of performance problems fall into a small number of patterns, and they are all diagnosable. This note covers the most common reasons queries slow down, how to use execution plans to see exactly what the database is doing, and the practical optimizations that fix most problems.
- **Prerequisite:** the [[Design]] folder, especially the notes on indexes. Understanding how indexes work is essential — optimization without that foundation is just guessing.

```ad-note
90% of database performance problems are solved by adding the right index. But the key word is "right." Profile first, optimize second — never guess which index to add.
```

---

## Why Queries Slow Down

- Almost every slow query traces back to one of these root causes. Learn to recognize them and you can fix most performance issues without deep DBA knowledge.

### 1. Missing Index — Full Table Scan

- When no index covers the columns in your `WHERE`, `JOIN`, or `ORDER BY` clause, the database has no choice but to read **every single row** in the table to find the ones you want. This is called a **full table scan** (or **clustered index scan** in SQL Server).
- On a 10-row table, this is instant. On a 10-million-row table, this can take minutes.

```sql
-- If there's no index on last_name, this scans every row:
SELECT first_name, last_name, email
FROM employees
WHERE last_name = 'Pham';
```

- The fix: create an index on the column being filtered.

```sql
CREATE INDEX IX_employees_last_name ON employees (last_name);
```

- After creating the index, the same query uses an **index seek** — it jumps directly to the matching rows instead of scanning everything.

```ad-important
A table scan on a small table (under ~1,000 rows) is often faster than using an index because the entire table fits in a single page read. The database optimizer knows this and may choose a scan even when an index exists. Don't force indexes on tiny tables.
```

### 2. SELECT * — Reading Unnecessary Columns

- `SELECT *` retrieves every column in the table, even if you only need two or three. This causes several problems:
  - **More data transferred** from disk to memory to network — wastes I/O bandwidth.
  - **Cannot use covering indexes** — the optimizer can't satisfy the query from the index alone if you're requesting columns that aren't in the index. It must go back to the base table (a **key lookup**), which is expensive.
  - **Fragile code** — if someone adds a large `VARBINARY` column to the table later, every `SELECT *` query suddenly pulls that data too.

```sql
-- BAD: pulls all 30 columns when you only need 3
SELECT * FROM orders WHERE customer_id = 42;

-- GOOD: pulls only what you need
SELECT order_id, order_date, total_amount
FROM orders
WHERE customer_id = 42;
```

### 3. N+1 Queries — One Query per Row Instead of One Query for All

- The **N+1 problem** is one of the most common performance killers in applications. It happens when your code runs one query to get a list of N items, then runs a separate query for each item to get related data.

```
-- Query 1: Get all 500 orders
SELECT order_id FROM orders WHERE status = 'pending';

-- Then for EACH order (500 separate queries!):
SELECT * FROM order_items WHERE order_id = 1;
SELECT * FROM order_items WHERE order_id = 2;
SELECT * FROM order_items WHERE order_id = 3;
... (497 more queries)
```

- This turns one round trip into 501 round trips. Each query has overhead: parsing, planning, execution, network latency. Multiply that by 500 and performance collapses.
- The fix is to use a **single query with a JOIN** or an `IN` clause:

```sql
-- ONE query instead of 501:
SELECT o.order_id, oi.product_id, oi.quantity, oi.price
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.status = 'pending';
```

```ad-note
N+1 problems are usually caused by ORM frameworks (Entity Framework, Hibernate) that lazily load related entities one at a time. In Entity Framework, use `.Include()` (eager loading) to avoid this. Always check the SQL your ORM generates.
```

### 4. No WHERE Clause — Reading the Entire Table

- A query without a `WHERE` clause returns every row in the table. If you're displaying data in a UI, you almost certainly don't need all 2 million rows. Use `WHERE` to filter and `TOP`/`LIMIT` to restrict the result set:

```sql
-- BAD: returns every row in a 2-million-row table
SELECT order_id, order_date, total_amount FROM orders;

-- GOOD: returns only what you need
SELECT order_id, order_date, total_amount
FROM orders
WHERE order_date >= '2026-01-01'
ORDER BY order_date DESC;
```

### 5. Implicit Conversions — Index Can't Be Used

- An **implicit conversion** occurs when you compare a column of one data type with a value of a different data type. The database must convert every row's value before comparing, which means it **cannot use an index** on that column.

```sql
-- Column phone_number is VARCHAR(20)
-- BAD: comparing VARCHAR to INT forces conversion of every row
SELECT * FROM customers WHERE phone_number = 5551234;

-- GOOD: compare with the correct type
SELECT * FROM customers WHERE phone_number = '5551234';
```

- This also happens with date columns, Unicode mismatches (`VARCHAR` vs `NVARCHAR` in SQL Server), and collation differences.

```ad-warning
Implicit conversions are especially dangerous because the query still *works* — it just runs 100x slower. There are no errors, no warnings in the result set. You only see the problem in the execution plan, where the optimizer shows a scan instead of a seek and a `CONVERT_IMPLICIT` warning.
```

---

## Execution Plans — How to SEE What the Database Is Doing

- An **execution plan** is the database's internal roadmap for how it will run your query. It shows every operation: which indexes it uses, how it joins tables, where it sorts, and how much work each step costs. Reading execution plans is the single most important skill for query optimization.

### Enabling Statistics (SQL Server)

- Before looking at graphical plans, you can get quick numeric feedback with statistics:

```sql
SET STATISTICS IO ON;    -- shows logical reads (pages read from memory/disk)
SET STATISTICS TIME ON;  -- shows CPU time and elapsed time

-- Now run your query:
SELECT first_name, last_name FROM employees WHERE department_id = 5;

-- Output will include:
-- Table 'employees'. Scan count 1, logical reads 3, physical reads 0
-- SQL Server Execution Times: CPU time = 0 ms, elapsed time = 1 ms.
```

- **Logical reads** is the most important number. It tells you how many 8KB data pages the database read to answer your query. Fewer reads = faster query. When comparing two versions of a query, the one with fewer logical reads is almost always better.

### Viewing Graphical Execution Plans in SSMS

- **Estimated plan:** press `Ctrl+L` — shows the plan without actually running the query. Good for expensive queries you don't want to execute yet.
- **Actual plan:** press `Ctrl+M` to enable, then run the query — shows the real plan with actual row counts. More accurate because it includes runtime statistics.
- **Live query statistics:** `Ctrl+Alt+S` while a query is running — shows the plan animating in real time with rows flowing through operators.

### Reading the Execution Plan

- The plan reads **right to left, top to bottom**. Data flows from right (sources) to left (final result). Each icon is an **operator** — a specific operation the engine performs.

#### Key Operators to Recognize

| Operator | Icon Shape | What It Means | Good or Bad? |
| --- | --- | --- | --- |
| **Table Scan** | Table with magnifying glass | Reads every row in a heap (table with no clustered index) | BAD on large tables |
| **Clustered Index Scan** | B-tree with magnifying glass | Reads every row via the clustered index | BAD on large tables (same as table scan but on a clustered table) |
| **Index Scan** | B-tree with magnifying glass | Reads every entry in a non-clustered index | Usually BAD — expected an index seek |
| **Index Seek** | B-tree with arrow | Jumps directly to matching entries in the index | GOOD — this is what you want |
| **Key Lookup** | Key icon | Found rows via a non-clustered index, but needs to go back to the clustered index to fetch additional columns | Acceptable in small quantities; problematic at scale |
| **RID Lookup** | Similar to key lookup | Same as key lookup but on a heap (no clustered index) | Same as key lookup — consider adding a clustered index |
| **Sort** | Arrows up/down | Sorts data in memory or on disk (tempdb) | Expensive for large sets — consider adding the sort column to an index |
| **Hash Match** | Hash symbol | Used for joins and aggregations on unsorted data | Often fine; can be expensive with large data sets |
| **Nested Loops** | Loop arrows | For each row in the outer input, scan the inner input | GOOD for small outer sets with indexed inner lookups; BAD for large sets |
| **Merge Join** | Merge arrows | Merges two pre-sorted inputs | Very efficient when both inputs are already sorted |

#### Operator Cost Percentages

- Each operator shows a **cost percentage** — what fraction of the total query cost that step represents. Look for the operators with the highest percentages first. A step showing 85% of the cost is where you should focus optimization.

```ad-note
Cost percentages are **estimates** based on statistics, not actual measurements. In estimated plans, they can be inaccurate — especially if table statistics are outdated. Always prefer the actual execution plan for critical analysis, and keep statistics up to date with `UPDATE STATISTICS`.
```

#### Thick vs Thin Arrows

- The **thickness of the arrows** between operators represents the number of rows flowing between them. A suddenly thick arrow means more rows than expected are being processed — often a sign of a bad join or missing filter.
- Hover over an arrow to see the **estimated vs actual row count**. A large discrepancy means the optimizer made a bad estimate, often due to stale statistics or parameter sniffing.

### Green Hints — Missing Index Suggestions

- SQL Server's execution plan will sometimes show a **green text message** at the top suggesting a missing index:

```
Missing Index Details from SQLQuery1.sql
The Query Processor estimates that implementing the following index
could improve the query cost by 95.23%.

CREATE NONCLUSTERED INDEX [IX_orders_customer_id]
ON [dbo].[orders] ([customer_id])
INCLUDE ([order_date], [total_amount])
```

- These suggestions are a good starting point but not gospel. The optimizer doesn't consider existing indexes, write overhead, or how many other queries use the table. Evaluate each suggestion before blindly creating it.

---

## Common Optimizations

### Only SELECT the Columns You Need

```sql
-- BAD:
SELECT * FROM customers WHERE city = 'Chicago';

-- GOOD:
SELECT customer_id, first_name, last_name
FROM customers
WHERE city = 'Chicago';
```

- This allows the optimizer to use a **covering index** — an index that contains all the columns the query needs, so it never has to touch the base table.

### Use EXISTS Instead of COUNT for Existence Checks

```sql
-- BAD: counts ALL matching rows when you only need to know if any exist
IF (SELECT COUNT(*) FROM orders WHERE customer_id = 42) > 0
    PRINT 'Customer has orders';

-- GOOD: stops as soon as it finds one matching row
IF EXISTS (SELECT 1 FROM orders WHERE customer_id = 42)
    PRINT 'Customer has orders';
```

- `COUNT(*)` must scan every matching row to compute the total. `EXISTS` short-circuits — it stops the moment it finds the first match. On a table with millions of matching rows, this difference is enormous.

### Use JOIN Instead of Correlated Subqueries When Possible

```sql
-- SLOWER: correlated subquery runs once for each row in the outer query
SELECT e.first_name, e.salary,
    (SELECT AVG(salary) FROM employees e2 WHERE e2.department_id = e.department_id) AS dept_avg
FROM employees e;

-- FASTER: JOIN with a derived table runs the aggregation once
SELECT e.first_name, e.salary, d.dept_avg
FROM employees e
JOIN (
    SELECT department_id, AVG(salary) AS dept_avg
    FROM employees
    GROUP BY department_id
) d ON e.department_id = d.department_id;
```

```ad-note
Modern query optimizers can often transform correlated subqueries into joins internally. But not always. When performance matters, write the join explicitly so you're not relying on the optimizer to figure it out.
```

### Add Indexes for WHERE, JOIN, and ORDER BY Columns

- The columns that benefit most from indexing are:
  1. Columns in `WHERE` clauses (filter conditions)
  2. Columns in `JOIN ON` conditions (join keys)
  3. Columns in `ORDER BY` clauses (sort order)
  4. Columns in `GROUP BY` clauses (aggregation keys)

- A single composite index can sometimes cover all of these:

```sql
-- Query:
SELECT order_id, order_date, total_amount
FROM orders
WHERE customer_id = 42 AND status = 'shipped'
ORDER BY order_date DESC;

-- Optimal index for this query:
CREATE INDEX IX_orders_cust_status_date
ON orders (customer_id, status, order_date DESC)
INCLUDE (total_amount);
```

- The `INCLUDE` clause adds columns to the **leaf level** of the index without including them in the sort key. This makes it a covering index without affecting the index's sort order or size as much.

### Avoid Functions on Indexed Columns in WHERE

- When you wrap an indexed column in a function, the database cannot use the index because it would need to evaluate the function on every row first (this is called making the predicate **non-SARGable** — Search ARGument able).

```sql
-- BAD (non-SARGable): index on Name is useless here
WHERE UPPER(Name) = 'LONG'

-- GOOD (SARGable): works with case-insensitive collation (CI)
WHERE Name = 'Long'

-- BAD (non-SARGable): index on order_date is useless
WHERE YEAR(order_date) = 2026

-- GOOD (SARGable): rewrite as a range
WHERE order_date >= '2026-01-01' AND order_date < '2027-01-01'

-- BAD (non-SARGable): index on phone is useless
WHERE LEFT(phone, 3) = '555'

-- GOOD (SARGable): use LIKE with a prefix
WHERE phone LIKE '555%'
```

```ad-important
The term **SARGable** (Search ARGument able) refers to a predicate that the query optimizer can satisfy using an index seek. A non-SARGable predicate forces a scan. Memorize this concept — it comes up constantly in performance tuning. The rule of thumb: the indexed column must be "naked" on one side of the comparison operator.
```

### Use Parameterized Queries for Plan Reuse

- When you send raw SQL with literal values, the database compiles a new execution plan for each unique query string:

```sql
-- These are THREE different queries to the optimizer, each needs its own plan:
SELECT * FROM orders WHERE customer_id = 1;
SELECT * FROM orders WHERE customer_id = 2;
SELECT * FROM orders WHERE customer_id = 3;
```

- With parameterized queries, the database compiles the plan once and reuses it:

```sql
-- One plan, reused for every customer_id value:
SELECT * FROM orders WHERE customer_id = @CustomerId;
```

- In C#, this is how you parameterize (see [[Parameterized Queries]] for full details):

```csharp
cmd.CommandText = "SELECT * FROM orders WHERE customer_id = @id";
cmd.Parameters.AddWithValue("@id", customerId);
```

- Plan reuse reduces CPU overhead and compilation time. On a busy server handling thousands of queries per second, this matters enormously.

```ad-warning
**Parameter sniffing** can occasionally cause problems with parameterized queries. The optimizer builds the plan based on the *first* parameter value it sees. If that value is atypical (e.g., a customer with 1 million orders when most have 10), the cached plan may be terrible for typical values. Solutions include `OPTION (RECOMPILE)` for specific queries, or `OPTIMIZE FOR` hints.
```

### Avoid SELECT DISTINCT as a Band-Aid

- If you're adding `DISTINCT` because your query returns duplicate rows, the problem is almost always an incorrect `JOIN` that's multiplying rows — typically a missing join condition or joining to a table with a one-to-many relationship you didn't account for.

```sql
-- BAD: masking a broken JOIN with DISTINCT
SELECT DISTINCT c.customer_name, c.email
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
-- Returns duplicates because each customer has many orders

-- GOOD: fix the query to return what you actually want
-- If you want customers who have orders:
SELECT c.customer_name, c.email
FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);
```

---

## Practical Examples — Good vs Bad Plans

### Example 1: Missing Index

```sql
-- Table: orders (5 million rows), no index on customer_id

-- Query:
SELECT order_id, order_date FROM orders WHERE customer_id = 42;
```

**Bad plan (no index):**
```
Clustered Index Scan (orders) → cost 100%
  Estimated rows: 5,000,000
  Actual rows returned: 47
  Logical reads: 28,450
```

**Good plan (after adding index):**
```sql
CREATE INDEX IX_orders_customer_id ON orders (customer_id) INCLUDE (order_date);
```
```
Index Seek (IX_orders_customer_id) → cost 100%
  Estimated rows: 47
  Actual rows returned: 47
  Logical reads: 3
```

- Logical reads dropped from 28,450 to 3 — a **9,483x improvement**.

### Example 2: Key Lookup Problem

```sql
-- Index exists on customer_id, but query also needs columns not in the index
SELECT order_id, order_date, total_amount, shipping_address
FROM orders
WHERE customer_id = 42;
```

**Plan with key lookups:**
```
Index Seek (IX_orders_customer_id) → 15%
  ↓
Key Lookup (clustered index) → 85%   ← THIS is the expensive part
  ↓
Nested Loops (join seek + lookup) → output
Logical reads: 189
```

**Fix: make the index a covering index:**
```sql
CREATE INDEX IX_orders_customer_id_covering
ON orders (customer_id)
INCLUDE (order_date, total_amount, shipping_address);
```

**Plan after covering index:**
```
Index Seek (IX_orders_customer_id_covering) → 100%
  No key lookup needed — all columns are in the index
Logical reads: 3
```

### Example 3: Non-SARGable Predicate

```sql
-- Index exists on hire_date

-- BAD: function on the indexed column
SELECT employee_id, first_name
FROM employees
WHERE YEAR(hire_date) = 2025;
-- Plan: Index SCAN (can't seek because of YEAR function)
-- Logical reads: 1,247

-- GOOD: rewrite as a range
SELECT employee_id, first_name
FROM employees
WHERE hire_date >= '2025-01-01' AND hire_date < '2026-01-01';
-- Plan: Index SEEK
-- Logical reads: 4
```

---

## Query Optimization Checklist

- When a query is slow, work through this checklist in order:

1. **Check the execution plan** — is it doing a scan where it should be doing a seek?
2. **Look at logical reads** — how many pages is it reading? Can you reduce that number?
3. **Check for missing indexes** — does the plan suggest one? Does the `WHERE`/`JOIN`/`ORDER BY` column have an index?
4. **Check for key lookups** — can you add `INCLUDE` columns to eliminate them?
5. **Check for implicit conversions** — are data types matching between the query and the column?
6. **Check for non-SARGable predicates** — are you wrapping indexed columns in functions?
7. **Check for N+1 patterns** — is your application sending hundreds of queries when one would do?
8. **Check the SELECT list** — are you selecting only the columns you need?
9. **Check for unnecessary DISTINCT** — is a bad join causing duplicate rows?
10. **Check statistics** — are they up to date? Run `UPDATE STATISTICS table_name` if not.

```ad-note
Don't optimize queries that aren't slow. Premature optimization wastes time and adds complexity (more indexes = slower writes). Use monitoring tools to identify your actual slow queries first, then apply this checklist to those specific queries.
```

---

**Next:** [[02 - Transactions and Concurrency]]
