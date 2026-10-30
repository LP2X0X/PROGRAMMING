---
tags: [sql, fundamentals, mysql]
---

- A **storage engine** is the component of a database server that handles **how data is physically stored, retrieved, updated, and locked**. It sits below the SQL layer — the server parses your query and optimizes it, then hands the actual read/write work to the storage engine.
- Think of it like this: the SQL server is the **waiter** (takes your order, understands the menu), and the storage engine is the **kitchen** (actually prepares the food). Same restaurant can have different kitchens.

---

### Why It Matters

The storage engine determines:

| Concern | Controlled by the engine |
|---|---|
| **Transactions** | Does the engine support `BEGIN`/`COMMIT`/`ROLLBACK`? |
| **Locking** | Row-level or table-level? See [[Locking and Versioning]] |
| **Crash recovery** | Can it recover uncommitted data after a crash? |
| **Foreign keys** | Are referential integrity constraints enforced? |
| **Full-text search** | Can it index and search text content? |
| **Storage format** | How data is laid out on disk (affects performance) |

Different engines make different trade-offs. You pick the engine based on your workload.

---

### MySQL's Storage Engine Architecture

MySQL is unique among major RDBMS because it has a **pluggable storage engine architecture** — you can choose a different engine per table. Most other databases (PostgreSQL, SQL Server, Oracle) have a single, tightly integrated storage layer that you don't swap out.

```sql
-- Check available engines
SHOW ENGINES;

-- Check what engine a table uses
SHOW TABLE STATUS WHERE Name = 'orders';

-- Create a table with a specific engine
CREATE TABLE logs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    msg TEXT
) ENGINE = InnoDB;

-- Change an existing table's engine (rewrites the entire table!)
ALTER TABLE logs ENGINE = MyISAM;
```

---

### InnoDB vs MyISAM

These are the two most well-known MySQL storage engines. **InnoDB** has been the default since MySQL 5.5 (2010).

| Feature | InnoDB | MyISAM |
|---|---|---|
| **Transactions** | Full ACID support | No transactions |
| **Locking** | Row-level locking | Table-level locking only |
| **Foreign keys** | Supported and enforced | Not supported |
| **Crash recovery** | Automatic via redo log (WAL) | No automatic recovery — tables can corrupt |
| **Concurrency** | High — row locks + MVCC | Low — entire table locked on write |
| **Full-text search** | Supported (since MySQL 5.6) | Supported |
| **Storage** | Clustered index (data stored with primary key) | Heap (data stored in insertion order) |
| **Count performance** | `SELECT COUNT(*)` scans the index | `SELECT COUNT(*)` returns instantly (stored count) |
| **Use case** | General purpose, transactional workloads | Read-heavy, legacy, non-critical data |

---

### Why InnoDB Won

Before MySQL 5.5, **MyISAM** was the default. It was faster for simple reads because it was simpler — no transaction overhead, no row locking, no crash recovery. But it had critical problems:

1. **No transactions** — you couldn't group operations. A crash mid-update left data half-written with no way to roll back.
2. **Table-level locking** — a single write blocked every other read and write on the entire table. Terrible for concurrent users.
3. **No foreign keys** — referential integrity had to be enforced in application code, leading to orphaned records.
4. **Crash corruption** — after an unclean shutdown, MyISAM tables often needed manual repair (`REPAIR TABLE`). Data loss was common.

InnoDB solved all of these. The performance gap narrowed as InnoDB matured, and the reliability advantages made it the obvious default.

```ad-warning
There is almost no reason to use MyISAM today. If you encounter it in an existing database, it's likely legacy. New tables should always use InnoDB.
```

---

### Other MySQL Engines

| Engine | Purpose |
|---|---|
| **MEMORY** (HEAP) | Stores data entirely in RAM. Extremely fast, but **all data is lost** on restart. Used for temporary tables or caches. |
| **CSV** | Stores data as comma-separated values files. No indexing. Useful for data exchange. |
| **ARCHIVE** | Optimized for storing large amounts of historical data with high compression. Insert-only (no `UPDATE` or `DELETE`). |
| **NDB (Cluster)** | Distributed storage engine for MySQL Cluster. High availability across multiple nodes. |
| **BLACKHOLE** | Accepts data but discards it (`/dev/null` for databases). Used in replication setups for filtering. |

---

### Other Databases

| Database | Storage approach |
|---|---|
| **PostgreSQL** | Single storage engine (heap-based) with pluggable access methods for indexes. No engine swapping per table. |
| **SQL Server** | Single integrated engine. Uses a page-based storage system. Row-store by default, with optional columnstore indexes for analytics. |
| **SQLite** | Single B-tree based engine. The entire database is one file. |
| **Oracle** | Single engine. Highly optimized with ASM (Automatic Storage Management) for disk management. |

These databases don't expose "storage engines" as a concept because their architecture is monolithic — everything is tightly integrated for consistency and optimization.

```ad-note
MySQL's pluggable engine architecture is both its strength and its quirk. It gives flexibility, but it means you need to be aware of which engine each table uses. In other databases, you don't think about this — it's handled internally. See [[Relational Databases]] for an overview of different RDBMS.
```
