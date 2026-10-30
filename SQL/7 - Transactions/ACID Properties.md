---
tags: [sql, transactions, term]
---

- **ACID** is a set of four properties that guarantee reliable database transactions. Understanding ACID is fundamental to working with any relational database.

---

### Atomicity

- A transaction is **all-or-nothing**. Either every operation in the transaction succeeds, or none of them do.
- If any part fails, the entire transaction is **rolled back** to its original state.

```
Transfer $500 from Account A to Account B:
  1. Debit Account A by $500
  2. Credit Account B by $500

If step 2 fails, step 1 is also undone. You never end up with money deducted but not credited.
```

---

### Consistency

- A transaction brings the database from one **valid state** to another valid state.
- All constraints ([[Primary Key]], [[Foreign Key]], [[Unique, Not Null, Check, Default|CHECK constraints]]) are respected. If a transaction would violate a constraint, it's rolled back.
- The database is never left in a "half-updated" state visible to other users.

---

### Isolation

- Concurrent transactions don't interfere with each other. Each transaction behaves as if it's the only one running.
- The degree of isolation is configurable via [[Isolation Levels]] — higher isolation means more safety but less concurrency.
- Without isolation, concurrent transactions can cause anomalies like dirty reads, non-repeatable reads, and phantom reads.

---

### Durability

- Once a transaction is **committed**, the changes are permanent — they survive power failures, crashes, and restarts.
- Databases achieve this through **write-ahead logging (WAL)** — every change is written to a log file on disk **before** being applied to the actual data files.

#### Crash Recovery

The WAL is what makes crash recovery possible. When a server crashes and restarts, it has two jobs:

1. **Roll forward committed transactions** — if a transaction was committed but the data hadn't been flushed from memory to disk yet, the server **replays the WAL** to reapply those changes. The commit was recorded in the log, so the data is recoverable.

2. **Roll back incomplete transactions** — if a transaction was in progress (no `COMMIT` issued) when the crash happened, the server **undoes all its changes**. The transaction never committed, so its modifications are discarded.

```
Server crashes mid-operation:

Transaction A: COMMIT issued, data in memory only → REDO from WAL (changes restored)
Transaction B: Still in progress, no COMMIT       → UNDO (changes discarded)
```

This is why the WAL is written **before** the actual data changes — if the server crashes after writing the log but before updating the data files, the log has everything needed to recover. If the log entry was never written, the transaction is treated as never having happened.

---

### Why ACID Matters

- **Financial systems**: a bank transfer must be atomic — debit and credit must both happen or neither.
- **E-commerce**: placing an order must deduct inventory and create the order record together.
- **Any multi-step operation**: updating related tables must be consistent, not half-done.

```ad-note
Not all databases are fully ACID-compliant. Some NoSQL databases sacrifice parts of ACID for performance or scalability. RDBMS like MySQL (InnoDB), PostgreSQL, and SQL Server are fully ACID-compliant.
```

- See [[BEGIN, COMMIT, ROLLBACK]] for how to use transactions in practice.
