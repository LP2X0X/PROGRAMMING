---
tags: [sql, transactions, term]
---

- A **transaction** is a logical unit of work — a group of one or more SQL statements that the database treats as a **single, indivisible operation**. Either all the statements succeed together, or none of them take effect.

---

### The Problem Transactions Solve

Without transactions, each SQL statement executes independently. If you need to make multiple related changes, a failure halfway through leaves the database in a **broken, half-updated state**:

```
❌ Without transactions:
  1. Debit $500 from Account A   → succeeds
  2. Credit $500 to Account B    → FAILS (server crash, constraint violation, etc.)
  
  Result: $500 vanished. Account A lost money, Account B never received it.
```

```
✅ With a transaction:
  1. BEGIN TRANSACTION
  2. Debit $500 from Account A
  3. Credit $500 to Account B    → FAILS
  4. ROLLBACK — undo step 2
  
  Result: nothing changed. Both accounts remain at their original balance.
```

The transaction groups the two operations into one atomic unit. If anything goes wrong at any point, the database **rolls back** all the changes made so far in that transaction, as if nothing happened.

---

### Transaction Lifecycle

Every transaction goes through a predictable lifecycle:

```
  BEGIN ──→ Execute statements ──→ COMMIT (success: changes saved permanently)
                │                         
                └─→ error or explicit ──→ ROLLBACK (failure: all changes undone)
```

1. **BEGIN** — marks the start of a transaction. From this point, changes are tentative.
2. **Execute statements** — `INSERT`, `UPDATE`, `DELETE`, etc. The database applies them but doesn't make them permanent yet. Other users may or may not see these changes depending on the [[Isolation Levels|isolation level]].
3. **COMMIT** — makes all changes permanent and visible to everyone. The transaction is complete.
4. **ROLLBACK** — undoes every change made since `BEGIN`. The database returns to its state before the transaction started.

See [[BEGIN, COMMIT, ROLLBACK]] for syntax and examples.

---

### Implicit (Auto-Commit) vs Explicit Transactions

#### Auto-Commit Mode (Default)

By default, every single SQL statement runs inside its **own invisible transaction** that auto-commits immediately:

```sql
-- This single INSERT is its own transaction
INSERT INTO logs (msg) VALUES ('hello');
-- Automatically committed — permanent immediately
```

This means if you run three `UPDATE` statements one after another (without `BEGIN`), each one is its own independent transaction. If the second one fails, the first one is **already committed** — you can't undo it.

#### Explicit Transaction

When you need multiple statements to succeed or fail as a group, you explicitly define the transaction boundary:

```sql
START TRANSACTION;
  UPDATE inventory SET stock = stock - 1 WHERE product_id = 42;
  INSERT INTO orders (customer_id, product_id) VALUES (7, 42);
COMMIT;
```

Now both statements are part of the same transaction. If the `INSERT` fails (maybe the customer doesn't exist), the `UPDATE` to inventory is also rolled back.

---

### What Can Go Inside a Transaction?

Transactions wrap **DML** (Data Manipulation Language) statements:

| Can be in a transaction | Cannot be in a transaction (most RDBMS) |
|---|---|
| `INSERT`, `UPDATE`, `DELETE` | `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE` |
| `SELECT` (for consistent reads) | `CREATE INDEX` |
| `MERGE` / `REPLACE` | `TRUNCATE` (in most RDBMS) |

**DDL statements** (`CREATE`, `ALTER`, `DROP`) cause an **implicit commit** in MySQL — they automatically commit any open transaction before executing. PostgreSQL is an exception: it supports **transactional DDL**, meaning you can `CREATE TABLE` inside a transaction and roll it back.

```sql
-- MySQL behavior:
START TRANSACTION;
  INSERT INTO logs (msg) VALUES ('test');
  CREATE TABLE temp (id INT);   -- ⚠️ implicit COMMIT happens here!
  -- The INSERT is now committed, cannot be rolled back
ROLLBACK;  -- too late, the INSERT is already permanent
```

---

### Transactions and Concurrency

When multiple users access the database at the same time, each runs their own transactions. This creates potential conflicts:

- **What if two people update the same row?** → one waits for the other's transaction to finish (see [[Locking and Versioning]])
- **What if one person reads data that another is modifying?** → the [[Isolation Levels|isolation level]] controls what they see
- **What if a long transaction holds resources?** → other transactions are blocked, reducing performance

This is why the general advice is: **keep transactions as short as possible**. Do your computation and validation before `BEGIN`, then wrap only the critical writes.

---

### Real-World Analogy

Think of a transaction like saving a document with multiple pages:

- **BEGIN** = you start editing
- **Your changes** = you modify pages 3, 7, and 12
- **COMMIT** = you hit Save — all changes are written to disk at once
- **ROLLBACK** = you hit Revert — the document goes back to how it was before you started editing

You'd never want a "save" that writes page 3 but crashes before saving page 7 — that's the half-updated state transactions prevent.

---

### Why Transactions Matter

| Scenario | Without transactions | With transactions |
|---|---|---|
| Bank transfer | Money can vanish mid-transfer | All-or-nothing: debit + credit happen together |
| E-commerce checkout | Order created but inventory not decremented | Both happen together or neither does |
| User registration | User row created but profile/settings table insert fails | User is fully created or not at all |
| Batch import | 500 of 1000 rows imported, then error | All 1000 imported or none |

```ad-note
Transactions are the foundation of reliable database operations. The guarantees they provide are formalized as [[ACID Properties]] — Atomicity, Consistency, Isolation, and Durability.
```
