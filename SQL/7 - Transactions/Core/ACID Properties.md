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
- Databases achieve this through write-ahead logging (WAL) — changes are written to a log on disk before being applied.

---

### Why ACID Matters

- **Financial systems**: a bank transfer must be atomic — debit and credit must both happen or neither.
- **E-commerce**: placing an order must deduct inventory and create the order record together.
- **Any multi-step operation**: updating related tables must be consistent, not half-done.

```ad-note
Not all databases are fully ACID-compliant. Some NoSQL databases sacrifice parts of ACID for performance or scalability. RDBMS like MySQL (InnoDB), PostgreSQL, and SQL Server are fully ACID-compliant.
```

- See [[BEGIN, COMMIT, ROLLBACK]] for how to use transactions in practice.
