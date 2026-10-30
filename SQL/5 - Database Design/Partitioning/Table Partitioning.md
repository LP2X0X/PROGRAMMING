---
tags:
  - sql
  - database-design
  - partitioning
  - performance
aliases:
  - Partitioning
  - Table Partitioning
  - Horizontal Partitioning
---

```ad-note
title: Overview
**Table partitioning** splits one large table into smaller physical pieces called **partitions**, while the database engine still presents it as a single logical table. Queries, inserts, and updates all work against the table name as usual — the engine silently routes operations to the correct partition(s). Partitioning is one of the most powerful tools for managing tables with millions or billions of rows, especially when combined with proper indexing and query design.
```

---

## Table of Contents

- [[#1. What is Partitioning]]
- [[#2. Horizontal vs Vertical Partitioning]]
- [[#3. Types of Partitioning]]
	- [[#3.1 RANGE Partitioning]]
	- [[#3.2 LIST Partitioning]]
	- [[#3.3 HASH Partitioning]]
	- [[#3.4 KEY Partitioning]]
	- [[#3.5 Composite (Sub-Partitioning)]]
- [[#4. Partition Pruning]]
- [[#5. Partition Management]]
- [[#6. Index Partitioning]]
- [[#7. Limitations and Trade-offs]]
- [[#8. Partitioning vs Manual Table Splitting]]
- [[#9. Practical Examples]]
- [[#10. Comprehensive Summary]]
- [[#Related Topics]]

---

## 1. What is Partitioning

- **Partitioning** divides a single large table into smaller, more manageable physical pieces called **partitions**. Each partition holds a subset of the table's rows based on a rule you define (a date range, a region code, a hash of the primary key, etc.).
- The key insight: ==from the application's perspective, nothing changes==. You still `SELECT`, `INSERT`, `UPDATE`, and `DELETE` against a single table name. The database engine transparently routes each operation to the correct partition(s) behind the scenes.

```ad-tip
title: C# Analogy
Think of partitioning like `Dictionary<string, List<Order>>` where the key is a year. All orders are logically "one collection," but physically you've split them into buckets by year. When you want orders from 2024, you go straight to the `"2024"` bucket instead of scanning every order. The database does exactly this — but automatically.
```

### Why Partition?

There are three main motivations:

| Motivation | What It Solves |
|---|---|
| **Query performance** | The optimizer can skip partitions that don't contain relevant rows (**partition pruning**), dramatically reducing I/O on large tables |
| **Maintenance efficiency** | Operations like `DROP PARTITION` remove millions of rows instantly (no row-by-row `DELETE`). `OPTIMIZE TABLE` can target a single partition instead of the whole table |
| **Data lifecycle management** | Old data can be archived or purged by simply dropping or detaching a partition — orders of magnitude faster than `DELETE WHERE date < ...` |

### When Does Partitioning Help?

- Tables with **millions+ rows** where queries naturally filter on a specific column (date, region, status).
- **Time-series data** (logs, audit trails, sensor readings) where old data is regularly purged.
- Tables where **maintenance windows** are tight — rebuilding an index on a 500 million row table takes hours; rebuilding one partition's index takes minutes.

### When Does Partitioning NOT Help?

- **Small tables** (under a few hundred thousand rows) — the overhead of partition management outweighs the benefit.
- Queries that **don't filter on the partition key** — without pruning, the engine scans *all* partitions, which is actually *slower* than scanning one unpartitioned table (due to per-partition overhead).
- Workloads that are **write-heavy with random access** — partitioning doesn't speed up random single-row lookups if the partition key isn't in the WHERE clause.

```ad-warning
title: Common Misconception
Partitioning is **not** a replacement for indexes. It's complementary. A partitioned table with no indexes still does full scans — just smaller ones (within each scanned partition). The real power comes from partitioning + indexes together: the engine first prunes to the relevant partition(s), then uses the index *within* that partition.
```

```ad-summary
title: Section Summary
- Partitioning splits a table into physical pieces while keeping it logically unified.
- The three main benefits are query performance (pruning), maintenance efficiency (per-partition operations), and data lifecycle management (instant drops).
- Partitioning does not replace indexes — they work together.
- It only helps when queries filter on the partition key; otherwise it adds overhead.
```

---

## 2. Horizontal vs Vertical Partitioning

There are two conceptual approaches to partitioning:

### Horizontal Partitioning (Row-Based)

- Splits the table **by rows**. Each partition contains a *subset of rows* but has the *same columns*.
- This is what `PARTITION BY` in MySQL/MariaDB does.
- Example: An `orders` table with 10 years of data is split so each partition holds one year's orders.

```
Full Table (10M rows, all columns):
┌──────────────────────────────────────┐
│  order_id | date       | amount | ...│
├──────────────────────────────────────┤
│  1        | 2020-01-15 | 99.99  | ...│  ── Partition p2020
│  2        | 2020-06-20 | 45.00  | ...│
├──────────────────────────────────────┤
│  3        | 2021-03-10 | 120.00 | ...│  ── Partition p2021
│  4        | 2021-11-05 | 75.50  | ...│
├──────────────────────────────────────┤
│  ...                                 │  ── Partition p2022, p2023, ...
└──────────────────────────────────────┘
```

### Vertical Partitioning (Column-Based)

- Splits the table **by columns**. Each piece has the *same rows* but a *subset of columns*.
- SQL engines don't natively support this via `PARTITION BY`. You achieve it manually by splitting a wide table into two or more narrower tables joined by a shared key.
- Useful when some columns are accessed frequently (name, status) while others are large/rarely used (BLOB, TEXT).

```
Original (wide):               Split into:
┌────────────────────────┐     ┌─────────────────┐  ┌───────────────────┐
│ id | name | bio (TEXT)  │     │ id | name        │  │ id | bio (TEXT)   │
│ 1  | Alice| <10KB text> │  →  │ 1  | Alice       │  │ 1  | <10KB text>  │
│ 2  | Bob  | <8KB text>  │     │ 2  | Bob         │  │ 2  | <8KB text>   │
└────────────────────────┘     └─────────────────┘  └───────────────────┘
                                 (hot columns)        (cold columns)
```

```ad-important
title: Focus of This Note
From here on, this note focuses exclusively on **horizontal partitioning** — the kind that MySQL/MariaDB's `PARTITION BY` clause implements. When people say "partitioning" in a MySQL context, they mean horizontal partitioning.
```

```ad-summary
title: Section Summary
- **Horizontal partitioning** splits by rows — each partition has the same columns but different rows. This is what MySQL's `PARTITION BY` does.
- **Vertical partitioning** splits by columns — done manually with separate tables and a shared key. Useful for wide tables with hot/cold columns.
- This note focuses on horizontal partitioning since that's the engine-native feature.
```

---

## 3. Types of Partitioning

MySQL/MariaDB supports four primary partitioning types, plus composite (sub-partitioning). Each type defines a different rule for deciding which partition a row belongs to.

| Type | Rule | Best For | Supports Pruning? |
|---|---|---|---|
| **RANGE** | Value falls within a range | Time-series data, sequential IDs | Yes |
| **LIST** | Value matches an explicit list | Categories, regions, status codes | Yes |
| **HASH** | Modulo of an expression | Even distribution when no natural range | Limited |
| **KEY** | MySQL's internal hash function | Even distribution using PK | Limited |
| **Composite** | Combination of two types | Large-scale systems needing two-level grouping | Depends on outer type |

---

### 3.1 RANGE Partitioning

The ==most common and most useful== type. Rows are assigned to partitions based on whether the partition key's value falls within a defined range.

**Syntax:**

```sql
CREATE TABLE orders (
    order_id    INT NOT NULL AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date  DATE NOT NULL,
    amount      DECIMAL(10,2),
    PRIMARY KEY (order_id, order_date)   -- partition key MUST be in PK
) PARTITION BY RANGE (YEAR(order_date)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION pmax  VALUES LESS THAN MAXVALUE
);
```

**How it works:**
- `YEAR(order_date)` is the **partition expression**. The engine evaluates it for each row.
- A row with `order_date = '2024-06-15'` evaluates to `2024`, which is `< 2025`, so it goes into `p2024`.
- `MAXVALUE` is a catch-all — any value not matched by earlier partitions goes here.

```ad-warning
title: Always Include a MAXVALUE Partition
Without `MAXVALUE`, inserting a row whose partition expression exceeds the highest boundary causes an error:
`Table has no partition for value 2027`
Always add a `MAXVALUE` catch-all partition, even if you plan to add new partitions regularly.
```

**RANGE COLUMNS variant:**

For partitioning directly on column values (especially `DATE`/`DATETIME`) without wrapping in a function:

```sql
CREATE TABLE orders (
    order_id    INT NOT NULL AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date  DATE NOT NULL,
    amount      DECIMAL(10,2),
    PRIMARY KEY (order_id, order_date)
) PARTITION BY RANGE COLUMNS (order_date) (
    PARTITION p2022 VALUES LESS THAN ('2023-01-01'),
    PARTITION p2023 VALUES LESS THAN ('2024-01-01'),
    PARTITION p2024 VALUES LESS THAN ('2025-01-01'),
    PARTITION p2025 VALUES LESS THAN ('2026-01-01'),
    PARTITION pmax  VALUES LESS THAN (MAXVALUE)
);
```

```ad-tip
title: RANGE COLUMNS vs RANGE
`RANGE COLUMNS` can partition on the raw column value (including `DATE`, `DATETIME`, `VARCHAR`, and multiple columns). Plain `RANGE` requires an expression that evaluates to an integer. Prefer `RANGE COLUMNS` for dates — it enables better partition pruning because the optimizer understands date comparisons directly, rather than needing to reason about `YEAR()` function results.
```

**Use cases:**
- Time-series tables (logs, orders, events) partitioned by year, month, or quarter.
- Sequential ID ranges (e.g., partition every 10 million IDs).
- Any scenario where you'll regularly drop old data.

---

### 3.2 LIST Partitioning

Rows are assigned based on the partition key matching a value in an **explicit list**. Think of it as a `CASE`/`switch` statement.

**Syntax:**

```sql
CREATE TABLE customers (
    customer_id INT NOT NULL AUTO_INCREMENT,
    name        VARCHAR(100),
    region      VARCHAR(20) NOT NULL,
    email       VARCHAR(255),
    PRIMARY KEY (customer_id, region)
) PARTITION BY LIST COLUMNS (region) (
    PARTITION p_na      VALUES IN ('US', 'CA', 'MX'),
    PARTITION p_europe  VALUES IN ('UK', 'DE', 'FR', 'ES', 'IT'),
    PARTITION p_asia    VALUES IN ('JP', 'KR', 'CN', 'IN', 'SG'),
    PARTITION p_other   VALUES IN ('AU', 'BR', 'ZA', 'OTHER')
);
```

**How it works:**
- A row with `region = 'US'` matches the `('US', 'CA', 'MX')` list, so it goes into `p_na`.
- Unlike RANGE, there is **no catch-all**. If you insert `region = 'RU'` and it's not in any list, you get an error.

```ad-danger
title: No MAXVALUE Equivalent for LIST
LIST partitioning has no `DEFAULT` or `MAXVALUE` catch-all in MySQL (MariaDB 10.2+ does support `DEFAULT`). You must ensure every possible value is accounted for. This is a common source of `INSERT` failures when a new region code is added but the partition definition isn't updated.
```

**Use cases:**
- Geographic regions, country codes.
- Status codes, category types.
- Any discrete set of values where you want per-group management.

---

### 3.3 HASH Partitioning

Distributes rows **evenly** across a fixed number of partitions using a modulo operation on an integer expression. You don't define the boundaries — you just say "give me N partitions" and the engine assigns rows by `expression MOD N`.

**Syntax:**

```sql
CREATE TABLE sessions (
    session_id  INT NOT NULL AUTO_INCREMENT,
    user_id     INT NOT NULL,
    created_at  DATETIME NOT NULL,
    data        TEXT,
    PRIMARY KEY (session_id)
) PARTITION BY HASH (session_id)
PARTITIONS 8;
```

**How it works:**
- A row with `session_id = 37` goes to partition `37 MOD 8 = 5` (partition number 5).
- Rows are spread roughly evenly across all 8 partitions.

**LINEAR HASH variant:**

```sql
PARTITION BY LINEAR HASH (session_id)
PARTITIONS 8;
```

- Uses a power-of-two algorithm instead of simple modulo.
- Advantage: adding/removing partitions only redistributes rows from one partition, not all of them.
- Disadvantage: slightly less even distribution.

```ad-note
title: When to Use HASH
HASH is useful when there's no natural range or list to partition on, but you still want to spread I/O across multiple physical files. It's less powerful than RANGE/LIST because the optimizer can't "prune" partitions as effectively — it can only prune when the WHERE clause uses `= constant` on the partition key (it knows exactly which partition `37 MOD 8` maps to), but range queries (`BETWEEN`, `>`, `<`) scan all partitions.
```

**Use cases:**
- Even I/O distribution when no natural grouping exists.
- Reducing lock contention by spreading hot rows across partitions.
- Tables where most queries do exact-match lookups on the partition key.

---

### 3.4 KEY Partitioning

Similar to HASH, but the engine uses its own **internal hash function** (MySQL uses `PASSWORD()` or MD5-based hashing) rather than a user-supplied expression. The key difference: KEY can partition on non-integer columns and automatically uses the primary key if no column is specified.

**Syntax:**

```sql
CREATE TABLE users (
    user_id    INT NOT NULL AUTO_INCREMENT,
    username   VARCHAR(50),
    email      VARCHAR(255),
    PRIMARY KEY (user_id)
) PARTITION BY KEY ()       -- empty = use the primary key
PARTITIONS 6;
```

Or specifying a column:

```sql
PARTITION BY KEY (username)
PARTITIONS 6;
```

**KEY vs HASH comparison:**

| Feature | HASH | KEY |
|---|---|---|
| Expression | User-defined (must return integer) | Engine's internal hash function |
| Column types | Integer only | Any column type (VARCHAR, DATE, etc.) |
| Default column | Must specify | Uses PRIMARY KEY if omitted |
| LINEAR variant | Yes | Yes |
| Distribution control | More predictable (you see the math) | Opaque (engine decides) |

```ad-tip
title: When to Prefer KEY Over HASH
Use KEY when your partition column isn't an integer (e.g., a UUID string or email) or when you just want the simplest "spread rows evenly" setup without thinking about the math. For integer columns where you want predictable placement, prefer HASH.
```

---

### 3.5 Composite (Sub-Partitioning)

Composite partitioning applies **two levels** of partitioning. The table is first split by one type (the outer partition), and each of those partitions is further split by a second type (the sub-partition).

MySQL supports these combinations:
- **RANGE–HASH** or **RANGE–KEY**
- **LIST–HASH** or **LIST–KEY**

You **cannot** sub-partition HASH by RANGE, or use RANGE within RANGE, etc. The outer type must be RANGE or LIST, and the inner type must be HASH or KEY.

**Syntax (RANGE–HASH):**

```sql
CREATE TABLE sensor_data (
    sensor_id   INT NOT NULL,
    reading_date DATE NOT NULL,
    value       DECIMAL(10,4),
    PRIMARY KEY (sensor_id, reading_date)
) PARTITION BY RANGE (YEAR(reading_date))
SUBPARTITION BY HASH (sensor_id)
SUBPARTITIONS 4 (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION pmax  VALUES LESS THAN MAXVALUE
);
```

**What this creates:**
- 4 RANGE partitions (by year) x 4 HASH sub-partitions (by sensor_id) = **16 physical sub-partitions**.
- A row with `reading_date = '2024-07-01'` and `sensor_id = 42` goes to: RANGE partition `p2024`, then sub-partition `42 MOD 4 = 2`.

```
sensor_data
├── p2023
│   ├── p2023_sp0  (sensor_id MOD 4 = 0)
│   ├── p2023_sp1  (sensor_id MOD 4 = 1)
│   ├── p2023_sp2  (sensor_id MOD 4 = 2)
│   └── p2023_sp3  (sensor_id MOD 4 = 3)
├── p2024
│   ├── p2024_sp0
│   ├── p2024_sp1
│   ├── p2024_sp2
│   └── p2024_sp3
├── p2025
│   └── ...
└── pmax
    └── ...
```

**Use cases:**
- Very large IoT/telemetry tables: RANGE by time (for easy purging) + HASH by device ID (for even I/O distribution within each time period).
- Multi-tenant systems: LIST by tenant + HASH by record ID.

```ad-warning
title: Sub-Partitioning Complexity
Sub-partitioning multiplies the number of physical partitions quickly (outer x inner). MySQL has a hard limit of **8192 partitions** per table (including sub-partitions). With 12 monthly RANGE partitions x 100 HASH sub-partitions, you'd have 1200 physical partitions — already a non-trivial amount of file descriptors and metadata.
```

```ad-summary
title: Section Summary
- **RANGE**: most common. Assigns rows by value ranges. Best for time-series and data lifecycle.
- **LIST**: assigns by explicit value lists. Best for categorical data (regions, statuses).
- **HASH**: even distribution via modulo. Best when no natural range/list exists.
- **KEY**: like HASH but uses the engine's internal hash. Works on non-integer columns.
- **Composite**: two-level partitioning (outer RANGE/LIST + inner HASH/KEY) for very large tables needing both time-based management and even distribution.
```

---

## 4. Partition Pruning

**Partition pruning** is the single most important performance benefit of partitioning. It's the optimizer's ability to ==identify and skip partitions that cannot contain matching rows== for a given query.

### How Pruning Works

When the optimizer analyzes a query's `WHERE` clause and finds conditions on the **partition key**, it determines which partitions could contain matching rows and only scans those. The rest are ignored entirely — no I/O, no page reads, nothing.

**Example:**

```sql
-- Table partitioned by RANGE (YEAR(order_date)) with partitions:
-- p2022, p2023, p2024, p2025, pmax

SELECT * FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

The optimizer sees that `YEAR('2024-01-01') = 2024` and `YEAR('2024-12-31') = 2024`, so ==only partition `p2024` is scanned==. Partitions `p2022`, `p2023`, `p2025`, and `pmax` are completely skipped.

If `orders` has 50 million rows across 5 years, the query only reads ~10 million rows (the 2024 partition) instead of all 50 million.

### Seeing Pruning with EXPLAIN

Use `EXPLAIN` to verify which partitions are being scanned:

```sql
EXPLAIN SELECT * FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

Output (simplified):

| id | select_type | table | partitions | type | rows |
|---|---|---|---|---|---|
| 1 | SIMPLE | orders | **p2024** | ALL | 10234567 |

The `partitions` column shows exactly which partitions the optimizer will scan. If you see **all partition names listed**, pruning is NOT happening — the optimizer couldn't determine which partitions to skip.

```ad-tip
title: EXPLAIN FORMAT=JSON for More Detail
For deeper analysis, use `EXPLAIN FORMAT=JSON` — it shows the optimizer's reasoning about partition selection in the `attached_condition` and `used_key_parts` fields.
```

### When Pruning Works

Pruning is effective when the `WHERE` clause contains conditions on the partition key that the optimizer can evaluate at plan time:

| Condition Type | Pruning? | Example |
|---|---|---|
| Equality on partition key | Yes | `WHERE order_date = '2024-06-15'` |
| Range on partition key | Yes | `WHERE order_date BETWEEN '2024-01-01' AND '2024-06-30'` |
| IN list on partition key | Yes | `WHERE YEAR(order_date) IN (2023, 2024)` |
| No condition on partition key | No | `WHERE customer_id = 42` |
| Function wrapping partition key | Maybe | Depends on whether the function matches the partition expression |
| OR with non-partition column | No | `WHERE order_date = '2024-01-01' OR amount > 100` |

### When Pruning Fails

```ad-danger
title: Pruning Killers
These common patterns **prevent** partition pruning, causing a full scan of all partitions:

1. **WHERE clause doesn't reference the partition key at all** — the optimizer has no information to prune on.
2. **Function mismatch** — if you partition by `RANGE (YEAR(order_date))` but query with `WHERE order_date > '2024-01-01'`, the optimizer may not realize that `YEAR('2024-01-01') = 2024`. Using `RANGE COLUMNS(order_date)` avoids this.
3. **OR conditions mixing partition and non-partition columns** — `WHERE order_date = '2024-01-01' OR amount > 100` can't prune because the `OR` means *any* partition could match via the `amount` condition.
4. **JOINs** — the driving table might be pruned, but the joined table is only pruned if its own WHERE condition allows it.
```

```ad-tip
title: Always EXPLAIN Partitioned Queries
After creating a partitioned table, run `EXPLAIN` on your most important queries and check the `partitions` column. If you see all partitions listed where you expected pruning, the partition key or query structure needs adjustment.
```

```ad-summary
title: Section Summary
- **Partition pruning** is the optimizer skipping irrelevant partitions based on WHERE clause conditions.
- It's the primary performance benefit of partitioning — turns a full-table scan into a scan of one or a few partitions.
- Use `EXPLAIN` to verify which partitions are scanned (check the `partitions` column).
- Pruning only works when the WHERE clause contains usable conditions on the partition key.
- Common pruning killers: missing partition key in WHERE, function mismatches, OR conditions, and poorly chosen partition keys.
```

---

## 5. Partition Management

One of the biggest operational advantages of partitioning is the ability to manage subsets of data independently. These `ALTER TABLE` operations work on individual partitions.

### Adding a New Partition

```sql
-- Before the MAXVALUE catch-all exists, just add:
ALTER TABLE orders ADD PARTITION (
    PARTITION p2026 VALUES LESS THAN (2027)
);
```

```ad-warning
title: Can't Add Before MAXVALUE
If a `MAXVALUE` partition exists, you can't `ADD PARTITION` because the new range would overlap. Instead, use `REORGANIZE PARTITION` to split the MAXVALUE partition. See [[#Reorganizing / Splitting Partitions]] below.
```

### Dropping a Partition (Instant Delete)

This is the **killer feature** for data lifecycle management:

```sql
ALTER TABLE orders DROP PARTITION p2022;
```

- This ==instantly removes all rows in partition `p2022`== — the engine simply drops the underlying data file.
- No row-by-row `DELETE`. No transaction log bloat. No long-running locks.
- Compare: `DELETE FROM orders WHERE YEAR(order_date) < 2023` on 10 million rows could take minutes to hours and generate massive redo logs. `DROP PARTITION` takes milliseconds.

```ad-tip
title: C# Analogy
Think of it like `Dictionary<int, List<Order>>`. If you want to remove all 2022 orders:
- Without partitioning: iterate the entire `List<Order>` and remove matching items one by one (`O(n)`).
- With partitioning: `dictionary.Remove(2022)` — the entire bucket is gone instantly (`O(1)`).
```

```ad-danger
title: DROP PARTITION Removes the Partition Definition Too
After `DROP PARTITION p2022`, the partition **definition** is also gone. If new 2022 data arrives (unlikely but possible), it would go into whatever partition now covers that range, or fail if no partition matches. Plan accordingly.
```

### Truncating a Single Partition

Like `DROP`, but keeps the partition definition:

```sql
ALTER TABLE orders TRUNCATE PARTITION p2022;
```

- Removes all rows but keeps the `p2022` partition ready to accept new data.
- Useful for resetting a partition without restructuring.

### Reorganizing / Splitting Partitions

Split an existing partition into two or more new ones:

```sql
-- Split pmax into a specific partition + new catch-all:
ALTER TABLE orders REORGANIZE PARTITION pmax INTO (
    PARTITION p2026 VALUES LESS THAN (2027),
    PARTITION pmax  VALUES LESS THAN MAXVALUE
);
```

- This physically moves rows from the old `pmax` into the appropriate new partitions.
- If `pmax` had accumulated 2026 data, those rows move into `p2026`; everything else stays in the new `pmax`.
- ==This operation is NOT instant== — it reads and rewrites the affected data.

### Merging Partitions

Combine two or more adjacent partitions into one:

```sql
ALTER TABLE orders REORGANIZE PARTITION p2022, p2023 INTO (
    PARTITION p_archive VALUES LESS THAN (2024)
);
```

- Merges the physical data from `p2022` and `p2023` into a single `p_archive` partition.
- Useful for consolidating old partitions that you want to keep but don't need granular access to.

### Exchanging a Partition with a Table

Swap a partition's data with a standalone table:

```sql
-- Create a matching non-partitioned table:
CREATE TABLE orders_2022_archive LIKE orders;
ALTER TABLE orders_2022_archive REMOVE PARTITIONING;

-- Swap:
ALTER TABLE orders EXCHANGE PARTITION p2022 WITH TABLE orders_2022_archive;
```

- After this, `orders_2022_archive` contains all the 2022 data, and `p2022` in `orders` is empty.
- This is a metadata-only operation (instant) — no data is physically copied.
- Extremely useful for archiving old partitions to separate tables.

```ad-tip
title: Archive Strategy
A common pattern for data lifecycle:
1. **Active**: data lives in partitioned `orders` table.
2. **Archive**: `EXCHANGE PARTITION` moves old data to an archive table.
3. **Purge**: eventually `DROP TABLE` the archive table.
This gives you fast queries on recent data while keeping old data accessible if needed.
```

### Summary of Management Operations

| Operation | Speed | Data Preserved? | Definition Preserved? |
|---|---|---|---|
| `ADD PARTITION` | Instant (metadata only) | N/A | N/A |
| `DROP PARTITION` | Instant | No | No |
| `TRUNCATE PARTITION` | Instant | No | Yes |
| `REORGANIZE PARTITION` | Slow (data rewrite) | Yes (moved) | Replaced |
| `EXCHANGE PARTITION` | Instant (metadata swap) | Yes (moved to other table) | Yes |

```ad-summary
title: Section Summary
- `DROP PARTITION` is the killer feature — instantly removes millions of rows with no DELETE overhead.
- `TRUNCATE PARTITION` empties a partition but keeps its definition.
- `REORGANIZE PARTITION` splits or merges partitions but requires a data rewrite (slow).
- `EXCHANGE PARTITION` swaps a partition's data with a standalone table (instant, great for archiving).
- `ADD PARTITION` fails if a MAXVALUE partition exists — use REORGANIZE instead.
```

---

## 6. Index Partitioning

Indexes on partitioned tables behave differently than on regular tables. Understanding this is critical for performance.

### Local Indexes (MySQL/MariaDB Default)

In MySQL/MariaDB, every index on a partitioned table is a **local index** — meaning ==each partition maintains its own independent copy of the index==.

```
orders (partitioned by year)
├── p2022
│   └── idx_customer_id (B-tree for p2022's rows only)
├── p2023
│   └── idx_customer_id (B-tree for p2023's rows only)
├── p2024
│   └── idx_customer_id (B-tree for p2024's rows only)
└── p2025
    └── idx_customer_id (B-tree for p2025's rows only)
```

**Implications:**
- An index lookup that targets one partition is fast — it uses that partition's local index.
- An index lookup that spans all partitions is slower — the engine queries each partition's index separately and merges results.
- Index rebuilds and maintenance can target individual partitions.

```sql
-- Rebuild index on just one partition:
ALTER TABLE orders REBUILD PARTITION p2024;

-- Analyze statistics for one partition:
ALTER TABLE orders ANALYZE PARTITION p2024;
```

### Global Indexes (Not in MySQL)

A **global index** is a single index spanning all partitions. When you query the index, it can point to rows in any partition.

```ad-important
title: MySQL Does NOT Support Global Indexes
Global indexes exist in **Oracle** and **PostgreSQL** (via global index types). MySQL/MariaDB only supports local indexes. This is one of the most significant limitations of MySQL partitioning.
```

**Why global indexes matter (and why their absence hurts):**
- With a local index, a query like `SELECT * FROM orders WHERE customer_id = 42` (where `customer_id` is NOT the partition key) must scan the `customer_id` index in *every* partition, because the engine doesn't know which partition(s) hold rows for customer 42.
- A global index would answer this in a single lookup.
- This is why **choosing the right partition key is critical** — it determines which queries benefit from pruning and which suffer from per-partition index scans.

### The PRIMARY KEY / UNIQUE Constraint Rule

```ad-danger
title: Critical MySQL Rule
In MySQL/MariaDB, ==the partition key (or all columns of the partition expression) must be included in every PRIMARY KEY and every UNIQUE index== on the table. This is a hard requirement enforced at table creation time.
```

**Why?** Because all indexes are local. A UNIQUE constraint on `(order_id)` alone can't be enforced across partitions — partition `p2022`'s local index has no knowledge of partition `p2024`'s rows. By requiring the partition key in the unique index, the engine can guarantee uniqueness *within* each partition, and since the partition key determines which partition a row belongs to, two rows with the same unique key must be in the same partition.

```sql
-- This FAILS:
CREATE TABLE orders (
    order_id   INT NOT NULL AUTO_INCREMENT,
    order_date DATE NOT NULL,
    PRIMARY KEY (order_id)             -- ERROR: partition key not in PK
) PARTITION BY RANGE (YEAR(order_date)) ( ... );

-- This WORKS:
CREATE TABLE orders (
    order_id   INT NOT NULL AUTO_INCREMENT,
    order_date DATE NOT NULL,
    PRIMARY KEY (order_id, order_date)  -- partition key included
) PARTITION BY RANGE (YEAR(order_date)) ( ... );
```

```ad-warning
title: Common Misconception
Many developers think adding the partition key to the PRIMARY KEY is just a "MySQL quirk" and doesn't matter. It has real consequences:
- The PK is now a **composite key**, which affects JOIN performance and foreign key design.
- `AUTO_INCREMENT` on `order_id` still works, but uniqueness is only guaranteed *within* a partition (though in practice, InnoDB's auto-increment counter is global, so duplicates are rare unless you're doing manual inserts).
- Any table referencing `orders` via a foreign key must reference the full composite key.
```

### Non-Unique Indexes

Non-unique indexes (regular `INDEX`) don't have the same restriction — they don't need to include the partition key. However, they are still local indexes, so a query using a non-unique index without filtering on the partition key will scan the index in every partition.

```ad-summary
title: Section Summary
- MySQL only supports **local indexes** — each partition has its own index copy.
- **Global indexes** (single index across all partitions) exist in Oracle/PostgreSQL but not MySQL.
- The partition key **must** be included in every PRIMARY KEY and UNIQUE index — a hard MySQL requirement.
- Non-unique indexes work without the partition key but are less efficient (scanned per-partition).
- Choosing a partition key that aligns with your most common queries is essential to avoid per-partition index scans.
```

---

## 7. Limitations and Trade-offs

### MySQL/MariaDB Limitations

| Limitation | Details |
|---|---|
| **Partition key in PK/UNIQUE** | Every unique index must include all columns of the partition expression |
| **Max partitions** | 8192 partitions per table (including sub-partitions) |
| **Foreign keys** | InnoDB partitioned tables **cannot** have foreign key constraints (neither referencing nor being referenced) in MySQL 5.7/8.0. MariaDB 10.5+ partially supports this |
| **Full-text indexes** | Not supported on partitioned InnoDB tables (MySQL 8.0 lifted this partially) |
| **Spatial indexes** | Not supported on partitioned tables |
| **TEMPORARY tables** | Cannot be partitioned |
| **Expression limitations** | `RANGE` requires integer-returning expressions (use `RANGE COLUMNS` for dates/strings). `HASH` also requires integer expressions |
| **No global indexes** | Every index is local — cross-partition unique lookups are expensive |

### When NOT to Partition

```ad-danger
title: Partitioning Anti-Patterns
1. **Small tables**: If your table has under ~1 million rows and fits comfortably in the buffer pool, partitioning adds complexity with no benefit.
2. **Wrong partition key**: If your most common queries don't filter on the partition key, you get *worse* performance (N partition scans instead of 1 table scan).
3. **"Partition by primary key just because"**: Partitioning by auto-increment ID with HASH doesn't help unless your queries filter by ID range.
4. **Assuming partitioning replaces indexing**: A partitioned table without indexes is just N smaller unindexed tables.
5. **Too many partitions**: Each partition has overhead (file descriptors, metadata, buffer pool entries). 1000+ partitions can degrade performance.
```

### The Partition Key Choice Problem

The partition key is the single most important decision in table partitioning. A bad choice negates all benefits:

| Scenario | Good Partition Key | Why |
|---|---|---|
| Log table queried by date range, old data purged monthly | `log_date` (RANGE by month) | Queries prune by date; `DROP PARTITION` handles purging |
| E-commerce orders queried by both customer and date | `order_date` (RANGE by year) | Most reports are date-based; customer lookups can use indexes within the date partition |
| User sessions table queried by session ID | Don't partition at all | Session lookups are random; no natural grouping to exploit |
| Multi-tenant SaaS with strict data isolation | `tenant_id` (LIST or HASH) | Every query includes tenant_id; pruning works naturally |

```ad-tip
title: Decision Rule
Ask yourself: **"Do 80%+ of my queries include this column in the WHERE clause?"** If yes, it's a good partition key candidate. If no, partitioning might hurt more than it helps.
```

```ad-summary
title: Section Summary
- MySQL has hard limitations: partition key must be in PK/UNIQUE, max 8192 partitions, no foreign keys on partitioned tables, no global indexes.
- Don't partition small tables, don't partition if queries don't filter on the partition key, and don't assume partitioning replaces indexes.
- The partition key is the most critical decision — it must align with your dominant query patterns.
```

---

## 8. Partitioning vs Manual Table Splitting

There are two approaches to splitting large tables: the engine's built-in `PARTITION BY` (covered above), and the **manual approach** of creating separate tables and stitching them together with a `UNION ALL` view. The manual approach is documented in [[Views]] under "Joining Partitioned Data."

### How Manual Splitting Works

You physically create separate tables and a view to unify them:

```sql
-- Separate tables:
CREATE TABLE orders_2022 ( ... );
CREATE TABLE orders_2023 ( ... );
CREATE TABLE orders_2024 ( ... );

-- Unifying view:
CREATE VIEW orders_all AS
    SELECT *, '2022' AS source_year FROM orders_2022
    UNION ALL
    SELECT *, '2023' AS source_year FROM orders_2023
    UNION ALL
    SELECT *, '2024' AS source_year FROM orders_2024;
```

### Comparison

| Feature | Engine Partitioning (`PARTITION BY`) | Manual Split + View (`UNION ALL`) |
|---|---|---|
| **Setup complexity** | One `CREATE TABLE` statement | Multiple tables + view + maintenance |
| **Transparency** | Fully transparent — app uses one table name | View looks like one table, but inserts/updates must target the real tables |
| **Insert routing** | Automatic — engine routes to correct partition | Manual — app must know which table to insert into |
| **Partition pruning** | Engine does it automatically via optimizer | Engine *may* prune `UNION ALL` branches if conditions are pushed down, but it's less reliable |
| **DROP old data** | `ALTER TABLE DROP PARTITION` — instant | `DROP TABLE orders_2022` — also instant |
| **Index management** | One set of index definitions (engine replicates per partition) | Separate index definitions per table (more flexible) |
| **Foreign keys** | Not supported on partitioned tables (MySQL) | Supported on each individual table |
| **Portability** | MySQL/MariaDB-specific syntax | Works on any SQL database |
| **Max flexibility** | Fixed partition types (RANGE, LIST, HASH, KEY) | Unlimited — any split logic you want |
| **Updatable** | Yes — full DML on the partitioned table | Views with `UNION ALL` are NOT updatable (read-only) |

### When to Use Each

```ad-tip
title: Decision Guide
- **Use engine partitioning** when:
  - You want fully transparent read/write operations.
  - You need automatic insert routing.
  - Your partitioning logic fits RANGE, LIST, HASH, or KEY.
  - You want optimizer-level pruning guarantees.

- **Use manual splitting + view** when:
  - You need foreign keys on the tables.
  - You need per-table storage engine differences.
  - Your partitioning logic is more complex than the four built-in types.
  - You want maximum portability across database engines.
  - You're already on a system that does this (legacy systems, data warehouses).
```

```ad-summary
title: Section Summary
- Engine partitioning is transparent (auto-routing, auto-pruning) but has MySQL-specific limitations (no FK, partition key in PK).
- Manual split + view is more flexible (supports FK, any split logic, portable) but requires manual insert routing and view maintenance.
- Engine partitioning for most cases; manual splitting for edge cases where MySQL's limitations are blocking.
- See [[Views]] under "Joining Partitioned Data" for the manual approach in detail.
```

---

## 9. Practical Examples

### Example 1: Partitioning a Log Table by Month

A classic use case — a rapidly growing `audit_log` table that needs fast queries on recent data and easy purging of old data.

```sql
CREATE TABLE audit_log (
    log_id      BIGINT NOT NULL AUTO_INCREMENT,
    user_id     INT NOT NULL,
    action      VARCHAR(50) NOT NULL,
    details     TEXT,
    created_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (log_id, created_at)       -- partition key in PK
) PARTITION BY RANGE COLUMNS (created_at) (
    PARTITION p2024_01 VALUES LESS THAN ('2024-02-01'),
    PARTITION p2024_02 VALUES LESS THAN ('2024-03-01'),
    PARTITION p2024_03 VALUES LESS THAN ('2024-04-01'),
    PARTITION p2024_04 VALUES LESS THAN ('2024-05-01'),
    PARTITION p2024_05 VALUES LESS THAN ('2024-06-01'),
    PARTITION p2024_06 VALUES LESS THAN ('2024-07-01'),
    PARTITION p2024_07 VALUES LESS THAN ('2024-08-01'),
    PARTITION p2024_08 VALUES LESS THAN ('2024-09-01'),
    PARTITION p2024_09 VALUES LESS THAN ('2024-10-01'),
    PARTITION p2024_10 VALUES LESS THAN ('2024-11-01'),
    PARTITION p2024_11 VALUES LESS THAN ('2024-12-01'),
    PARTITION p2024_12 VALUES LESS THAN ('2025-01-01'),
    PARTITION pmax     VALUES LESS THAN (MAXVALUE)
);
```

### Example 2: Dropping Old Data Instantly

Purge January 2024 logs — millions of rows removed in milliseconds:

```sql
-- Without partitioning (slow):
DELETE FROM audit_log WHERE created_at < '2024-02-01';
-- Takes minutes. Generates massive redo log. Locks rows.

-- With partitioning (instant):
ALTER TABLE audit_log DROP PARTITION p2024_01;
-- Done in milliseconds. No row-by-row delete. Minimal locking.
```

### Example 3: Adding a New Month's Partition

When a new month approaches, split the MAXVALUE catch-all:

```sql
ALTER TABLE audit_log REORGANIZE PARTITION pmax INTO (
    PARTITION p2025_01 VALUES LESS THAN ('2025-02-01'),
    PARTITION pmax     VALUES LESS THAN (MAXVALUE)
);
```

```ad-tip
title: Automate This
In production, you'd schedule a monthly job (cron, MySQL Event Scheduler, or application-level) that:
1. Creates next month's partition (REORGANIZE pmax).
2. Drops the oldest partition if retention policy says so.
This keeps the partition maintenance completely hands-off.
```

### Example 4: Verifying Partition Pruning with EXPLAIN

```sql
-- Query that SHOULD prune:
EXPLAIN SELECT * FROM audit_log
WHERE created_at BETWEEN '2024-06-01' AND '2024-06-30'\G
```

Expected output:

```
           id: 1
  select_type: SIMPLE
        table: audit_log
   partitions: p2024_06          <-- only one partition scanned
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1523456
     filtered: 100.00
        Extra: Using where
```

```sql
-- Query that WON'T prune (no partition key in WHERE):
EXPLAIN SELECT * FROM audit_log
WHERE user_id = 42\G
```

Expected output:

```
           id: 1
  select_type: SIMPLE
        table: audit_log
   partitions: p2024_01,p2024_02,p2024_03,...,pmax   <-- ALL partitions!
         type: ALL
         rows: 18000000
        Extra: Using where
```

The second query scans every partition because `user_id` is not the partition key. If this is a common query, you'd want an index on `user_id` (which would be a local index within each partition) or reconsider the partitioning strategy.

### Example 5: Partitioned Table with Secondary Index

```sql
CREATE TABLE audit_log (
    log_id      BIGINT NOT NULL AUTO_INCREMENT,
    user_id     INT NOT NULL,
    action      VARCHAR(50) NOT NULL,
    details     TEXT,
    created_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (log_id, created_at),
    INDEX idx_user (user_id),                -- local index per partition
    INDEX idx_action (action, created_at)    -- composite, includes partition key
) PARTITION BY RANGE COLUMNS (created_at) (
    ...
);
```

The `idx_action` index includes `created_at` (the partition key), so queries like `WHERE action = 'LOGIN' AND created_at > '2024-06-01'` get both pruning AND index usage — the best of both worlds.

```ad-summary
title: Section Summary
- Monthly RANGE COLUMNS partitioning is the standard pattern for log/audit tables.
- `DROP PARTITION` replaces slow, locking `DELETE` operations for data purging.
- `REORGANIZE PARTITION` splits the MAXVALUE catch-all to add new months.
- Always verify pruning with `EXPLAIN` — check the `partitions` column.
- Combine partitioning with secondary indexes for optimal query performance.
```

---

## 10. Comprehensive Summary

```ad-tip
title: Complete Summary
**Table partitioning** splits a single large table into smaller physical pieces (partitions) while keeping it logically unified. The database engine handles routing reads and writes to the correct partition(s) transparently.

**Why partition:**
- **Performance**: partition pruning lets the optimizer skip irrelevant partitions entirely, dramatically reducing I/O for queries that filter on the partition key.
- **Maintenance**: per-partition operations (`DROP`, `TRUNCATE`, `REBUILD`, `ANALYZE`) work on subsets of data instead of the whole table.
- **Data lifecycle**: `DROP PARTITION` instantly removes millions of rows — no row-by-row DELETE, no transaction log bloat, no long locks.

**Five partitioning types:**
- **RANGE** (most common): by value ranges, ideal for time-series data.
- **LIST**: by explicit value lists, ideal for categorical data.
- **HASH**: by modulo for even distribution when no natural grouping exists.
- **KEY**: like HASH but using the engine's internal hash, works on non-integer columns.
- **Composite**: two-level (RANGE/LIST outer + HASH/KEY inner) for very large tables.

**Critical rules in MySQL:**
- The partition key must be in every PRIMARY KEY and UNIQUE index.
- All indexes are local (per-partition) — no global indexes.
- Foreign keys cannot reference or be defined on partitioned tables.
- Max 8192 partitions (including sub-partitions) per table.

**Partition pruning** is the main performance benefit. The optimizer reads the WHERE clause and only scans partitions that could contain matching rows. Always verify with `EXPLAIN`.

**vs. Manual splitting**: engine partitioning is transparent and automatic; manual split + UNION ALL view (see [[Views]]) is more flexible but requires manual insert routing and is read-only through the view.

**Key decision rule**: only partition if 80%+ of your queries filter on the partition key. Wrong key choice = no pruning = worse performance than no partitioning.
```

---

## Related Topics

- [[Views]] — manual table splitting with UNION ALL views (see "Joining Partitioned Data" section)
- [[What are Indexes]] — how indexes work, trade-offs, B-tree fundamentals
- [[ALTER TABLE]] — the base DDL command used in partition management operations
- [[DROP and TRUNCATE]] — non-partitioned equivalents of DROP/TRUNCATE PARTITION
- [[Primary Key]] — why the partition key must be included in the PK
- [[Normal Forms]] — normalization principles that inform table design decisions
- [[When to Denormalize]] — denormalization trade-offs, related to partitioning as a performance strategy
