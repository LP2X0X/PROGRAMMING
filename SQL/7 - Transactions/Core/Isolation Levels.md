---
tags: [sql, transactions]
---

- Isolation levels control how much transactions can "see" each other's uncommitted or concurrent changes. Higher isolation = more safety but less concurrency (more locking/blocking).

---

### Transaction Anomalies

| Anomaly               | What happens                                                |
| --------------------- | ----------------------------------------------------------- |
| **Dirty read**        | Reading data that another transaction hasn't committed yet. If that transaction rolls back, you read data that never existed. |
| **Non-repeatable read** | Reading the same row twice in one transaction gets different values because another transaction modified and committed it between reads. |
| **Phantom read**      | Running the same query twice returns different rows because another transaction inserted or deleted rows that match your filter. |

---

### The Four Isolation Levels (least → most strict)

#### READ UNCOMMITTED
- Can see uncommitted changes from other transactions (**dirty reads** allowed).
- Fastest, but rarely used — the data you read may be rolled back.
```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

#### READ COMMITTED
- Only sees data that has been **committed**. Prevents dirty reads.
- **Default** in PostgreSQL and SQL Server.
- Non-repeatable reads and phantom reads are still possible.
```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

#### REPEATABLE READ
- Once you read a row, re-reading it in the same transaction returns the **same values**.
- **Default** in MySQL/InnoDB.
- Prevents dirty reads and non-repeatable reads. Phantom reads are still possible in standard SQL (but MySQL/InnoDB prevents them too using gap locks).
```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

#### SERIALIZABLE
- Transactions execute as if they were **sequential** (one after another).
- Prevents all anomalies. Safest but **slowest** — uses heavy locking or MVCC validation.
```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

---

### Summary Table

| Level              | Dirty Read | Non-Repeatable Read | Phantom Read |
| ------------------ | ---------- | ------------------- | ------------ |
| READ UNCOMMITTED   | Possible   | Possible            | Possible     |
| READ COMMITTED     | Prevented  | Possible            | Possible     |
| REPEATABLE READ    | Prevented  | Prevented           | Possible*    |
| SERIALIZABLE       | Prevented  | Prevented           | Prevented    |

*MySQL/InnoDB also prevents phantom reads at REPEATABLE READ level.

---

### Practical Guidance

- **Most applications**: READ COMMITTED is sufficient and the best balance of safety and performance.
- **Financial/critical operations**: use SERIALIZABLE or at minimum REPEATABLE READ for the specific transaction.
- You can set isolation per-transaction — you don't have to use the same level everywhere.

```ad-warning
Higher isolation levels increase lock contention and can cause **deadlocks** (two transactions waiting for each other's locks). Keep transactions short and access tables in a consistent order to minimize deadlocks. See [[BEGIN, COMMIT, ROLLBACK]].
```
