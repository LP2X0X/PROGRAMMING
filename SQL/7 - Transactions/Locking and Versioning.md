---
tags: [sql, transactions, concurrency]
---

- When multiple users access the same data simultaneously, the database needs a strategy to prevent conflicts. There are two main approaches: **locking** and **versioning**.

---

### Strategy 1: Read and Write Locks

- **Writers** must request a **write lock** to modify data
- **Readers** must request a **read lock** to query data
- Multiple readers can hold read locks simultaneously (reading doesn't conflict with reading)
- Only **one write lock** is given out at a time per table (or portion of it)
- Read requests are **blocked** until the write lock is released

```
User A: writing row 5   → holds write lock
User B: wants to read   → BLOCKED, waits for User A to finish
User C: wants to write   → BLOCKED, waits for User A to finish
```

**Trade-off:** safe but slow — readers wait on writers, writers wait on other writers.

---

### Strategy 2: Versioning (MVCC)

- **Writers** must still request a **write lock**
- **Readers don't need any lock** — they read freely at all times
- The database ensures each reader sees a **consistent snapshot** of the data from the moment their query began, even if other users are modifying data during that time

```
User A: writing row 5   → holds write lock
User B: wants to read   → reads the OLD version of row 5, no waiting
User C: wants to write   → BLOCKED, waits for User A
```

This is called **MVCC (Multi-Version Concurrency Control)** — the database keeps multiple versions of each row so readers always have a consistent view without waiting.

**Trade-off:** faster for read-heavy workloads (readers never block), but the database uses more storage to maintain row versions.

---

### Lock Granularity

A lock can be applied at different levels. The server decides **how much data** to lock at once. The smaller the scope, the more concurrency you get — but the database has to track more individual locks, which costs memory and CPU.

---

#### Table Locks

- Locks the **entire table** — no other user can modify any row in that table until the lock is released
- Other users wanting to write to **any row** in the table must wait
- Very low overhead for the server (one lock to manage) but very low concurrency (everyone waits)
- Used for bulk operations like `ALTER TABLE`, `TRUNCATE`, or large batch imports where locking every row individually would be wasteful

```
Table: employees (1000 rows)
User A: UPDATE employees SET salary = 50000 WHERE id = 5
        → TABLE LOCK acquired
User B: UPDATE employees SET name = 'Bob' WHERE id = 999
        → BLOCKED — even though it's a completely different row
```

---

#### Page Locks

- A **page** is a fixed-size chunk of memory (typically 2 KB to 16 KB depending on the database) that holds multiple rows
- Locks all the rows on that **one page**, but rows on other pages are free
- A middle ground between table locks and row locks — more concurrency than table locks, less overhead than row locks
- SQL Server uses page locks; MySQL/InnoDB does not (it goes straight from row to table)

```
Table: employees (1000 rows, spread across 50 pages)

Page 1: rows 1-20
Page 2: rows 21-40
Page 3: rows 41-60

User A: modifying row 5  → locks Page 1 (rows 1-20)
User B: modifying row 25 → locks Page 2 — no conflict, proceeds
User C: modifying row 10 → BLOCKED — row 10 is on Page 1, which User A holds
```

---

#### Row Locks

- Locks only the **specific row** being modified — all other rows in the table remain free
- Maximum concurrency — multiple users can modify different rows in the same table simultaneously
- Highest overhead — the server must track a separate lock for each row being modified, which uses more memory
- This is the **default** for most modern databases (MySQL/InnoDB, PostgreSQL, Oracle, SQL Server for DML)

```
Table: employees (1000 rows)

User A: UPDATE employees SET salary = 50000 WHERE id = 5
        → locks row 5 only
User B: UPDATE employees SET name = 'Bob' WHERE id = 999
        → locks row 999 — no conflict, proceeds immediately
User C: UPDATE employees SET salary = 60000 WHERE id = 5
        → BLOCKED — row 5 is locked by User A
```

---

#### Lock Escalation

When the server holds **too many row-level locks** (hundreds or thousands), it may automatically **escalate** to a broader lock (page or table) to reduce overhead. This is a trade-off the database makes internally:

```
User A: UPDATE employees SET salary = salary * 1.1 WHERE department = 'Sales'
        → starts locking individual rows: row 3, row 7, row 12, row 15...
        → too many row locks → server escalates to TABLE LOCK
        → simpler for the server to manage, but blocks everyone else
```

The threshold for escalation varies by database. You rarely control this directly — it's an internal optimization.

---

#### Summary

| Granularity | Scope | Concurrency | Overhead | Used when |
|---|---|---|---|---|
| **Table lock** | Entire table | Lowest | Lowest | DDL operations, bulk imports |
| **Page lock** | 2–16 KB chunk of rows | Medium | Medium | SQL Server default for some operations |
| **Row lock** | Single row | Highest | Highest | Default for most DML (`INSERT`, `UPDATE`, `DELETE`) |

---

### Comparison

| | Read/Write Locks | Versioning (MVCC) |
|---|---|---|
| **Readers block on writers?** | Yes | No |
| **Writers block on writers?** | Yes | Yes |
| **Multiple simultaneous readers?** | Yes | Yes |
| **Consistency** | Via locks — only one accessor at a time | Via snapshots — each reader sees a frozen-in-time view |
| **Performance** | Slower for mixed read/write workloads | Better for read-heavy workloads |
| **Storage** | No extra versions needed | Must store old row versions |
| **Used by** | SQL Server (default), DB2 | MySQL/InnoDB, PostgreSQL, Oracle |

```ad-note
MySQL's InnoDB engine uses MVCC by default. This is why readers don't block writers in MySQL — each reader gets a snapshot. The versioning behavior ties directly to [[Isolation Levels]] — different levels control how "fresh" that snapshot is.
```
