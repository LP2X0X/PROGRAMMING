---
tags:
 - csharp
 - database
 - ado-net
---

## Why Databases Are Hard in Practice

Working with databases in development is straightforward — small data, one user, local machine. Production is a different world. These are the problems every team faces.


---

## 1. Concurrency — Multiple Users at Once

When multiple users read and write the same data simultaneously, things go wrong:

```
User A: reads balance = $100
User B: reads balance = $100
User A: withdraws $80  → writes $20
User B: withdraws $80  → writes $20   ← should fail, but both saw $100

Result: $160 withdrawn from a $100 account.
```

This is a **race condition**. Databases solve it with:
- **Locks** — block other users while one is writing
- **Transactions** — group operations so they succeed or fail together
- **Isolation levels** — control how much concurrent users can see each other's changes

Getting the balance right between safety and performance is the core challenge. Too much locking → slow. Too little → data corruption.

See [[Transactions]] for isolation levels and how to use them in C#.


---

## 2. Performance at Scale

Queries that work fine in development collapse under real data:

| Data size | Reality |
|---|---|
| 100 rows | Any query is fast, doesn't matter |
| 1 million rows | Need indexes, query tuning |
| 100 million rows | Need partitioning, caching, read replicas |
| 1 billion rows | Need sharding, distributed systems, dedicated DBAs |

A query that takes 1ms on dev data can take 30 seconds in production. Common causes:
- Missing indexes — full table scan on millions of rows
- N+1 queries — one query per item instead of one query for all items
- SELECT * — reading 50 columns when you need 3
- No pagination — loading 100,000 rows into memory

```ad-note
The most impactful performance fix is almost always adding the right index. Understanding execution plans is a critical skill.
```


---

## 3. Schema Changes on Live Data

Altering a table with millions of rows is risky:

```sql
ALTER TABLE Users ADD Email NVARCHAR(255)
```

Questions you must answer:
- Does this lock the entire table? For how long?
- What happens to queries running right now?
- Does the app code expect the old schema or new schema?
- Do you deploy the code first or the schema first?
- Can you roll back if it fails halfway?

Every schema change on a production database risks **downtime** or **data corruption**. This is why migration tools (EF Core Migrations, Flyway, DbUp) exist — they version and track schema changes.


---

## 4. Data Integrity

Preventing invalid data from entering the database:

| Problem | Example |
|---|---|
| Orphaned records | Order references a customer that was deleted |
| Duplicates | Same user registered twice with different casing |
| Invalid state | Negative inventory, future birth dates |
| Encoding issues | Emoji stored as `???` because of wrong charset |
| Null confusion | `NULL` vs empty string `""` vs the text `"NULL"` |

Constraints (foreign keys, unique, check) help, but they can't catch everything. Application-level validation is also needed.


---

## 5. Backup and Recovery

```
"The server crashed. When was the last backup?"

Full backup: Sunday night
Today: Friday
5 days of data: GONE
```

Worse scenarios:
- Backups were running, but **nobody tested restoring them** — files are corrupted
- Backups exist, but they're on the **same disk** that failed
- Backup restores take 8 hours — business is down the entire time

```ad-warning
A backup you haven't tested restoring is not a backup. Regularly test your restore process.
```


---

## 6. Security

| Threat | What happens |
|---|---|
| SQL injection | Attacker executes arbitrary SQL through your app |
| Overprivileged accounts | App connects as admin — one bug exposes everything |
| Unencrypted connections | Passwords sent in plain text over the network |
| Sensitive data in logs | Credit card numbers in error messages |
| Unauthorized access | Who has access to production data? |

SQL injection is still the **#1 web vulnerability** year after year. Always use [[Parameterized Queries]] — never concatenate user input into SQL.


---

## 7. Migrations and Deployments

```
Version 1: Users table has a Name column
Version 2: Split into FirstName and LastName
```

How do you:
1. Deploy the schema change?
2. Migrate 10 million existing rows?
3. Keep the app running during migration?
4. Roll back if something goes wrong?
5. Handle old code that still reads `Name`?

A common strategy is **expand-contract**:
1. **Expand** — add new columns, keep old ones, write to both
2. **Migrate** — backfill new columns from old data
3. **Contract** — remove old columns after all code uses the new ones


---

## 8. Connection Management

```csharp
// Developer forgets to dispose:
var conn = new SqlConnection(connStr);
conn.Open();
// ... forgot conn.Close() or using

// After 100 requests → pool exhausted → app hangs
// "Timeout expired. All pooled connections were in use."
```

One leaked connection can kill an entire application. This is why `using` is critical — and why [[Connection Pooling]] exists.

```ad-important
ALWAYS use `using` with database connections. A single leaked connection under load will exhaust the pool and hang your app.
```


---

## 9. Replication and Consistency

```
User writes: "Update email to new@email.com"  → goes to PRIMARY
User reads:  "What's my email?"                → reads from REPLICA
Result:      "old@email.com"                   ← stale! Replica hasn't caught up yet
```

This is **eventual consistency** — the replica will catch up, but there's a delay (milliseconds to seconds). Users see stale data and get confused.

Solutions: read-after-write consistency, sticky sessions, or accept the delay with proper UX ("changes may take a moment to appear").


---

## 10. The Human Factor

The most dangerous database tool is a developer with production access:

```sql
-- Meant to delete 2014 orders:
DELETE FROM Orders WHERE Year = 2024
--                              ^^^^  one digit, all 2024 data gone

-- Forgot the WHERE clause:
UPDATE Users SET IsAdmin = 1
-- Every user is now admin
```

Mitigations:
- **Read-only production access** for most developers
- **Require WHERE clause** in UPDATE/DELETE (some tools enforce this)
- **Point-in-time backups** — restore to just before the mistake
- **Review process** for production SQL changes


---

## Summary

| Problem | Core difficulty |
|---|---|
| Concurrency | Multiple users reading/writing same data simultaneously |
| Performance | Queries that work on small data fail at scale |
| Schema changes | Altering live tables with millions of rows without downtime |
| Data integrity | Preventing invalid, duplicate, or orphaned data |
| Backup/recovery | Proving you can actually restore when disaster hits |
| Security | SQL injection, access control, encryption |
| Migrations | Changing schema without downtime or data loss |
| Connections | Leaked connections exhaust the pool and hang the app |
| Replication | Read replicas lag behind writes |
| Human error | One wrong WHERE clause deletes everything |

```ad-note
This is why "database administrator" is an entire career — not just a task. In small teams, developers handle all of this. In large organizations, dedicated DBAs manage production databases while developers work with local/staging copies.
```


---

## See Also

- [[Database and Server Mental Model]] — how servers, databases, and clients fit together
- [[ADO.NET Overview]] — how your C# code connects to databases
- [[Parameterized Queries]] — preventing SQL injection
- [[Transactions]] — handling concurrency with isolation levels
- [[Connection Pooling]] — why connection management matters
