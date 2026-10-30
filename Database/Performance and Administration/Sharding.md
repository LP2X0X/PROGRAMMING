---
tags:
 - database
 - performance
 - sharding
 - distributed-systems
 - architecture
aliases:
 - Database Sharding
 - Horizontal Sharding
---

```ad-summary
title: Overview
**Database sharding** is the technique of splitting a single logical database across multiple independent database servers, where each server holds a *different subset* of the data. It is the most powerful — and most complex — approach to horizontal scaling. This note covers shard key selection, sharding strategies, the hard problems sharding introduces, routing mechanisms, how sharding compares to other scaling techniques, real-world examples, and application-level concerns for C#/.NET developers. Sharding is a last resort: understand everything you lose before you gain the ability to scale beyond a single machine.
```

---

## Table of Contents

- [[#What Is Sharding]]
- [[#Shard Key — The Most Critical Decision]]
- [[#Sharding Strategies]]
- [[#The Hard Problems of Sharding]]
- [[#Routing — How the App Finds the Right Shard]]
- [[#Sharding vs Partitioning vs Clustering vs Replication]]
- [[#When to Shard — The Progression]]
- [[#Real-World Examples]]
- [[#C# and Application-Level Considerations]]
- [[#Limitations and Trade-offs Summary]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## What Is Sharding

- **Sharding** means splitting a large database into smaller, independent pieces called **shards**, where each shard lives on its own database server and holds a *different subset* of the rows.
- Each shard is a **fully independent database** — it has its own tables, indexes, connections, storage, and query engine. It knows nothing about the other shards.
- The collection of all shards together represents the complete dataset. No single shard has all the data. If you need *all* users, you must ask *every* shard.

```ad-info
title: Definition — Shard
A **shard** is one partition of the overall dataset, hosted on its own database server instance. It contains a subset of the rows (e.g., users 1–1,000,000) but has the *same schema* as every other shard. Think of it as one volume of an encyclopedia — same structure, different content.
```

### Sharding vs Other Scaling Approaches

People often confuse sharding with similar-sounding techniques. The critical distinction:

| Technique | Where Data Lives | What Each Server Has |
|---|---|---|
| **Partitioning** | One server | Same table, split into pieces on disk |
| **Replication** | Multiple servers | Every server has the *same* data |
| **Clustering** | Multiple servers | Shared disk / shared data across nodes |
| **Sharding** | Multiple servers | Each server has *different* data |

- **Partitioning** ([[Table Partitioning]]) splits a table into segments on a *single* server. The application doesn't know the table is partitioned — it queries normally and the database routes to the right partition internally. No network hops, no distributed complexity.
- **Replication** copies all data to read replicas. Every replica has the full dataset. Great for read scaling, useless for write scaling — writes still go to one primary.
- **Clustering** (e.g., MySQL NDB Cluster, Galera) keeps all nodes in sync with the same data. Adds availability and some read scaling. Still limited by the size of a single dataset.
- **Sharding** is the only approach where each server holds *different* rows. This is what lets you scale *both* reads and writes beyond a single machine's capacity — but at enormous complexity cost.

```ad-tip
title: The Analogy
Think of a library. **Partitioning** is one library with books organized into sections. **Replication** is building identical copies of the whole library. **Clustering** is one library with multiple checkout counters. **Sharding** is splitting the books across multiple buildings — fiction at Library A, non-fiction at Library B — and you need to know which building to visit for the book you want.
```

### Why Shard At All?

Sharding solves problems that no single-server solution can:

1. **Dataset too large for one server** — when your data exceeds the storage capacity of the biggest machine you can buy (or afford).
2. **Write throughput ceiling** — replication scales reads, but all writes funnel through one primary. Sharding distributes writes across N servers.
3. **Query latency at scale** — even with perfect indexes, a table with billions of rows has slower operations than a table with millions. Smaller tables = faster queries.
4. **Geo-distribution** — place shards near users. European users hit the EU shard, US users hit the US shard. Lower latency for everyone.

```ad-summary
title: Section Summary
- Sharding splits data across multiple independent database servers, each holding different rows.
- Unlike partitioning (one server), replication (same data), or clustering (shared data), sharding distributes *different* subsets of data to each node.
- It enables scaling beyond a single machine for both reads and writes.
- Sharding is motivated by dataset size, write throughput limits, query latency, or geo-distribution needs.
```

---

## Shard Key — The Most Critical Decision

- The **shard key** is the column (or combination of columns) that determines which shard a given row belongs to.
- Every row in the table is assigned to exactly one shard based on its shard key value.
- ==This is the single most important decision in a sharding architecture.== Get it wrong, and you will rebuild the entire system. Get it right, and sharding becomes manageable.

```ad-info
title: Definition — Shard Key
The **shard key** is the column used to compute which shard stores a given row. For example, if `user_id` is the shard key, then `hash(user_id) MOD 4` might determine which of four shards holds that user's data.
```

### The Golden Rule

```ad-important
title: The Rule
**Shard by the entity your queries center around.** If 90% of your queries include `WHERE user_id = ?`, then `user_id` is your shard key. The goal is that the vast majority of queries can be answered by a *single* shard without consulting the others.
```

### Properties of a Good Shard Key

A good shard key has these characteristics:

| Property | Why It Matters |
|---|---|
| **High cardinality** | Many distinct values means data can be distributed finely across shards. A column with only 3 values can only fill 3 shards. |
| **Even distribution** | Values should be roughly equally common. If 80% of rows have the same value, 80% of data lands on one shard — you've just created a bigger version of the original problem. |
| **Query alignment** | Most queries filter on this column, so they can be routed to a single shard. If queries rarely use the shard key, every query becomes a scatter-gather across all shards. |
| **Immutability** | The value should rarely or never change. If a user's `region` changes, you have to physically move their rows between shards — a complex, error-prone operation. |
| **Non-monotonic** | Values shouldn't always increase. If the shard key is auto-increment, every new row goes to the same "last" shard, creating a write hotspot. |

### Good Shard Key Examples

| Shard Key | Why It Works |
|---|---|
| `user_id` | High cardinality, queries center on users, immutable. Most SaaS apps query by user. |
| `tenant_id` | For multi-tenant SaaS: all data for one tenant on one shard. Queries almost always filter by tenant. Natural isolation. |
| `region` / `country_code` | For geo-distributed systems: keeps user data close to them. Works when queries are regional. |
| `merchant_id` | E-commerce/payments: all transactions for a merchant on one shard. Queries are merchant-centric. |

### Bad Shard Key Examples

```ad-danger
title: Shard Keys That Will Ruin Your Architecture
These seem reasonable but cause serious problems:
```

| Shard Key | Problem |
|---|---|
| `created_at` (timestamp) | **Hotspot.** All new writes go to the "current time" shard. Yesterday's shard sits idle while today's shard is on fire. Range-based sharding on time is a recipe for unbalanced load. |
| `status` (enum) | **Too few values.** If `status` can only be `active`, `inactive`, `pending`, you can only have 3 shards. And most rows are probably `active`, so one shard gets almost everything. |
| `auto_increment id` | **Monotonic.** All new rows have the highest ID, so all writes go to the last shard. Same hotspot problem as timestamp. Hash-based sharding on auto-increment can mitigate this, but range-based will fail. |
| `email` | **Rarely queried.** If your app mainly queries by `user_id`, sharding by `email` means every user lookup is a scatter-gather across all shards. Shard key must match query patterns. |
| `order_id` | **Splits related data.** If orders and order_items shard differently, joining them requires cross-shard queries. Better to shard both by `customer_id` so a customer's orders and items land together. |

### Compound Shard Keys

- Sometimes no single column works well. You can use a **compound shard key** — a combination of columns:

```sql
-- Shard by (tenant_id, region)
-- Routes data based on both tenant AND region
-- Enables single-shard queries that filter on both columns
```

- Compound keys give finer control but add complexity. Any query that doesn't include *all* parts of the compound key becomes a scatter-gather.

```ad-warning
title: Common Misconception — "Just Shard by Primary Key"
Sharding by primary key (usually auto-increment) seems logical but is often the *worst* choice. The primary key exists for uniqueness, not for query routing. Your shard key should reflect how your *application queries data*, not how the database identifies rows.
```

```ad-summary
title: Section Summary
- The shard key determines which shard holds each row — it is the most critical decision in a sharding architecture.
- Good shard keys have high cardinality, even distribution, align with query patterns, are immutable, and are non-monotonic.
- Shard by the entity your queries center around (user_id, tenant_id, etc.).
- Bad shard keys create hotspots (timestamps, auto-increment), uneven distribution (enums), or scatter-gather on every query (columns not used in WHERE clauses).
- Compound shard keys offer finer control at the cost of more complex routing.
```

---

## Sharding Strategies

Three fundamental approaches exist for mapping a shard key value to a specific shard. Each has trade-offs.

### 1. Range-Based Sharding

- Divide the shard key's value space into contiguous ranges, and assign each range to a shard.

```
Shard 1: user_id     1 – 1,000,000
Shard 2: user_id 1,000,001 – 2,000,000
Shard 3: user_id 2,000,001 – 3,000,000
Shard 4: user_id 3,000,001 – 4,000,000
```

```sql
-- Routing logic (conceptual):
-- If user_id BETWEEN 1 AND 1000000      → connect to shard_1
-- If user_id BETWEEN 1000001 AND 2000000 → connect to shard_2
-- etc.

-- On shard_1:
SELECT * FROM users WHERE user_id = 42;
-- Fast! Row is guaranteed to be on this shard.

-- Range query within one shard:
SELECT * FROM orders WHERE user_id BETWEEN 500 AND 1000;
-- Also fast — all these users are on shard_1.
```

**Advantages:**
- Simple to understand and implement
- **Range queries within a shard are efficient** — `WHERE user_id BETWEEN X AND Y` hits one shard if the range falls within its boundaries
- Easy to add new shards: just extend the range space (Shard 5: 4,000,001+)
- Data locality: related sequential values are on the same shard

**Disadvantages:**
- **Hotspots.** If the shard key is monotonically increasing (auto-increment, timestamp), all new writes go to the last shard. The other shards sit idle.
- **Uneven distribution.** The ranges may not contain equal amounts of data. If 90% of active users are in the 1–1M range, Shard 1 is overloaded.
- Requires manual rebalancing when shards become uneven — split ranges, migrate data.

```ad-warning
title: The Timestamp Trap
Range-based sharding on `created_at` is one of the most common mistakes. All new data goes to the "newest" shard, creating an ever-growing hotspot. If you must shard by time, combine it with another key (e.g., `tenant_id + date_bucket`) or use hash-based sharding instead.
```

### 2. Hash-Based Sharding

- Apply a hash function to the shard key, then use modulo to determine the shard.

```
shard_number = hash(shard_key) MOD number_of_shards
```

```sql
-- Conceptual routing:
-- shard = hash(user_id) MOD 4
-- hash(42)    MOD 4 = 2  → Shard 2
-- hash(43)    MOD 4 = 0  → Shard 0
-- hash(1000)  MOD 4 = 1  → Shard 1
-- hash(1001)  MOD 4 = 3  → Shard 3

-- On shard_2:
SELECT * FROM users WHERE user_id = 42;
-- App computed hash(42) MOD 4 = 2, connected to shard_2, runs query.
```

**Advantages:**
- **Even distribution.** A good hash function spreads data uniformly across shards, regardless of the input pattern. No hotspots.
- **No range bias.** Sequential IDs get scattered across shards, so monotonically increasing keys are not a problem.
- Simple to compute — one hash + one modulo operation.

**Disadvantages:**
- **Range queries are impossible.** `WHERE user_id BETWEEN 100 AND 200` could hit *every* shard because the hash scatters those IDs across all shards. You must query all shards and merge.
- **Resharding is painful.** When you add a shard (change N from 4 to 5), `hash(key) MOD 4` and `hash(key) MOD 5` produce different results for most keys. Millions of rows must be physically moved. This is the biggest operational headache with hash-based sharding.

```ad-example
title: The Resharding Problem
With 4 shards: `hash(42) MOD 4 = 2` → row is on Shard 2.
You add a 5th shard: `hash(42) MOD 5 = 2` → still Shard 2 (lucky).
But `hash(43) MOD 4 = 0` vs `hash(43) MOD 5 = 3` → must move from Shard 0 to Shard 3.

On average, when going from N to N+1 shards, **~(N-1)/N of all rows** must be remapped. Going from 4 to 5 shards means ~75% of your data might need to move. With billions of rows, this is a multi-day migration.
```

#### Consistent Hashing — The Mitigation

- **Consistent hashing** solves the resharding problem by mapping both shards *and* keys onto a virtual ring (hash space 0 to 2^32).
- Each shard owns a segment of the ring. A key is assigned to the next shard clockwise from its position on the ring.
- When you add a shard, it takes over a segment of the ring. Only the keys in *that segment* need to move — typically ~1/N of the data, not (N-1)/N.

```
            0
         /     \
       /         \
      |  Shard A  |
       \         /
        \       /
   Shard C --- Shard B

Adding Shard D only affects the segment it takes over
from one neighbor — not the entire dataset.
```

```ad-tip
title: Consistent Hashing in Practice
Consistent hashing is used by Amazon DynamoDB, Apache Cassandra, and most distributed key-value stores. If you're implementing sharding from scratch, consistent hashing is almost always worth the extra complexity over simple `MOD N`. Libraries exist for every major language including C# (`HashRing` implementations on NuGet).
```

### 3. Directory-Based Sharding

- A **lookup table** (directory) explicitly maps each shard key value (or range of values) to a specific shard. Every query first asks the directory "where is this key?", then connects to the indicated shard.

```sql
-- The directory table (on a dedicated metadata server):
CREATE TABLE shard_directory (
    shard_key_value BIGINT PRIMARY KEY,
    shard_id        INT NOT NULL,
    shard_host      VARCHAR(255) NOT NULL
);

-- Lookup:
SELECT shard_host FROM shard_directory WHERE shard_key_value = 42;
-- Returns: 'shard-02.db.internal'
-- App connects to that host and runs the actual query.

-- Moving a user to a different shard is just an UPDATE:
UPDATE shard_directory
SET shard_id = 5, shard_host = 'shard-05.db.internal'
WHERE shard_key_value = 42;
```

**Advantages:**
- **Maximum flexibility.** You can place any key on any shard. Move individual users, rebalance shards one record at a time, handle VIP customers differently.
- **No resharding formula.** Adding a shard just means updating directory entries to point to the new shard. No hash recalculation.
- **Heterogeneous shards.** Different shards can have different capacities — the directory routes more traffic to bigger machines.

**Disadvantages:**
- **Single point of failure.** The directory is queried on *every* database operation. If it goes down, the entire system is unreachable.
- **Performance bottleneck.** Every query adds a directory lookup round trip. Must be cached aggressively (with cache invalidation complexity).
- **Additional infrastructure.** You now need a highly-available, low-latency directory service (often implemented as a replicated in-memory database like Redis or etcd).

```ad-danger
title: Directory Availability
The directory service must be **more reliable** than any individual shard. If Shard 3 goes down, you lose 25% of users. If the directory goes down, you lose *everyone* because no query can be routed. Plan for replication, failover, and caching of the directory.
```

### Strategy Comparison

| Aspect | Range-Based | Hash-Based | Directory-Based |
|---|---|---|---|
| **Distribution** | Uneven (depends on data) | Even (hash guarantees it) | Controlled (you decide) |
| **Range queries** | Efficient within a shard | Impossible (scatter-gather) | Depends on how directory is structured |
| **Hotspot risk** | High (monotonic keys) | Low (hash scatters) | Low (manually balanced) |
| **Resharding** | Manual range splitting | Painful (MOD N changes); consistent hashing helps | Easy (update directory entries) |
| **Complexity** | Low | Medium | High (directory infrastructure) |
| **Flexibility** | Low | Low | High (per-key routing) |
| **Single point of failure** | None | None | Directory service |
| **Best for** | Non-monotonic, range-query workloads | High-throughput, even-distribution needs | Systems needing fine-grained control |

```ad-summary
title: Section Summary
- **Range-based**: simple, enables range queries, but prone to hotspots with monotonic keys.
- **Hash-based**: even distribution, no hotspots, but kills range queries and makes resharding expensive. Consistent hashing mitigates resharding cost.
- **Directory-based**: maximum flexibility and easy rebalancing, but introduces a single point of failure in the lookup service.
- No strategy is universally best — choose based on your query patterns and operational needs.
```

---

## The Hard Problems of Sharding

Sharding introduces a set of distributed systems problems that do not exist in a single-database architecture. These are the reasons sharding is a last resort.

### 1. Cross-Shard Queries (Scatter-Gather)

- When a query cannot be routed to a single shard (because it doesn't filter on the shard key, or spans multiple shard key values), it must be sent to **every shard**, and the results must be merged.
- This is called the **scatter-gather pattern**: scatter the query to all shards, gather and combine the results.

```sql
-- This query includes the shard key → single shard:
SELECT * FROM orders WHERE user_id = 42;
-- App knows user 42 is on Shard 2. Fast.

-- This query does NOT include the shard key → scatter-gather:
SELECT * FROM orders WHERE order_date > '2026-01-01';
-- Must query ALL shards, collect results, merge.
-- Latency = slowest shard's response time.
```

```ad-warning
title: Scatter-Gather Performance
Scatter-gather queries are limited by the **slowest shard**. If 3 of 4 shards respond in 10ms but one takes 500ms, the whole query takes 500ms. This is called the **tail latency problem** and gets worse as you add more shards. Any performance outlier on any shard becomes the bottleneck.
```

### 2. Cross-Shard JOINs

- JOINs across shards are **not possible at the database level.** The database engine on Shard 1 has no knowledge of the data on Shard 2. You cannot write:

```sql
-- THIS DOES NOT WORK IN A SHARDED SYSTEM:
SELECT u.name, o.total
FROM shard1.users u
JOIN shard2.orders o ON u.user_id = o.user_id;
-- Each shard is an independent database — they can't see each other.
```

**Workarounds:**
1. **Co-locate related data.** Shard `users` and `orders` by the same key (`user_id`), so a user's orders are on the same shard as their profile. JOINs work within one shard.
2. **Application-level JOIN.** Query both shards separately, then join the results in your application code. Slower, uses more memory, complex to paginate.
3. **Denormalize.** Copy frequently joined data into the table that needs it (e.g., store `user_name` directly in the `orders` table). Trades storage and update complexity for query simplicity.
4. **Materialized views / data warehouse.** ETL data from all shards into a single read-only aggregate database for analytics queries.

```ad-important
title: Co-Location Is the Key Strategy
The most effective mitigation is to shard related tables by the same key so that JOINs within a shard remain possible. If `users`, `orders`, and `order_items` are all sharded by `user_id`, then all of User 42's data lives on the same shard, and JOINs work normally within that shard.
```

### 3. Resharding / Adding Shards

- When your shards outgrow their capacity, you need to add more. This means changing the mapping from key → shard, which means **physically moving data** between servers.
- With hash-based sharding (`hash(key) MOD N`), changing N from 4 to 5 remaps approximately 75% of all keys. Each remapped key's rows must be read from the old shard, inserted into the new shard, and deleted from the old shard — while the system remains operational.

**Approaches to resharding:**

| Approach | How It Works | Trade-off |
|---|---|---|
| **Stop-the-world** | Take the system offline, migrate data, bring it back up | Simplest but requires downtime |
| **Double-write** | Write to both old and new shard during migration; read from old until migration complete; switch reads to new; stop writing to old | Zero downtime but complex and doubles write load |
| **Shadow reads** | During migration, read from old shard; periodically verify against new shard; switch when verified | Catches data inconsistencies but slow |
| **Consistent hashing** | Only ~1/N of data moves when adding a shard | Best approach; standard in modern systems |

```ad-danger
title: Resharding Is the Hardest Operational Problem in Sharding
Live resharding — migrating data while the system serves traffic — is one of the hardest problems in distributed systems. You must handle: concurrent writes to migrating rows, application routing during the transition (old mapping vs new mapping), data verification after migration, rollback if something goes wrong, and monitoring that no data is lost. Companies like Stripe have written extensively about how resharding takes months of engineering effort.
```

### 4. Distributed Transactions

- In a single database, a transaction guarantees atomicity: all statements succeed or all fail. When a transaction spans two shards, no single database can enforce this guarantee.
- The standard solution is **two-phase commit (2PC)**:

```
Phase 1 — PREPARE:
  Coordinator → Shard 1: "Can you commit?"
  Coordinator → Shard 2: "Can you commit?"
  Both respond: "Yes, I'm prepared."

Phase 2 — COMMIT:
  Coordinator → Shard 1: "Commit."
  Coordinator → Shard 2: "Commit."
  Both commit.

If either shard says "No" in Phase 1:
  Coordinator → All shards: "Abort."
```

**Problems with 2PC:**
- **Slow.** Multiple network round trips between coordinator and participants.
- **Blocking.** If the coordinator crashes after Phase 1 but before Phase 2, the participants are stuck holding locks, waiting for a decision that may never come.
- **Reduces throughput.** Locks are held longer because the transaction can't commit until all shards agree.

```ad-tip
title: Avoiding Distributed Transactions
The best approach is to design your shard key so that transactions *don't* cross shards. If all of User 42's data is on one shard, transferring between their accounts is a local transaction. Cross-shard transactions arise when you need to atomically update data on different shards — minimize these by co-locating related data.
```

**Alternative: Saga Pattern**
- Instead of 2PC, use a **saga** — a sequence of local transactions with compensating actions if a step fails:
  1. Shard A: debit $100 from Account A (local commit)
  2. Shard B: credit $100 to Account B (local commit)
  3. If step 2 fails: Shard A: credit $100 back to Account A (compensating transaction)
- Sagas provide **eventual consistency** rather than strong consistency. The system may be temporarily inconsistent between steps.

### 5. Auto-Increment ID Collisions

- In a single database, `AUTO_INCREMENT` guarantees unique IDs. With multiple shards, each shard has its own auto-increment counter, producing duplicate IDs across shards.

```sql
-- Shard 1 creates user with id = 1
-- Shard 2 creates user with id = 1
-- Two different users, same ID! Which is which?
```

**Solutions:**

| Solution | How It Works | Trade-off |
|---|---|---|
| **Offset + Increment** | Shard 1: 1, 3, 5, 7... Shard 2: 2, 4, 6, 8... (each shard uses `auto_increment_increment = N`, `auto_increment_offset = shard_number`) | Simple but must reconfigure all shards when adding one. Gaps in IDs. |
| **UUID / GUID** | Generate `UUID()` for each row. Globally unique by design. | 128-bit = 36 chars as string. Bad for indexes (random, no locality). Storage-heavy. |
| **Snowflake IDs** | 64-bit IDs encoding: timestamp + machine ID + sequence number. Twitter invented this. | Unique, sortable, compact. Requires a Snowflake ID generator service or library. |
| **Central ID service** | One service (e.g., a Redis `INCR`) generates all IDs. | Simple IDs but the service is a single point of failure and potential bottleneck. |
| **ULID** | Universally Unique Lexicographically Sortable Identifier. Like UUID but time-ordered. | Sortable (unlike UUID v4), 128-bit, growing ecosystem support. |

```sql
-- MySQL: Configuring offset and increment per shard

-- On Shard 1:
SET GLOBAL auto_increment_increment = 4;  -- total number of shards
SET GLOBAL auto_increment_offset = 1;     -- this shard's number
-- Produces: 1, 5, 9, 13, 17, ...

-- On Shard 2:
SET GLOBAL auto_increment_increment = 4;
SET GLOBAL auto_increment_offset = 2;
-- Produces: 2, 6, 10, 14, 18, ...
```

```ad-tip
title: Snowflake IDs Are the Industry Standard
Most modern distributed systems use Snowflake-style IDs: 64-bit integers that encode a timestamp, a machine/shard identifier, and a sequence number. They are unique across all shards, roughly time-sortable, and compact enough for efficient indexing. Libraries exist for every major language — in C#, look for `IdGen` on NuGet.
```

### 6. Referential Integrity

- **Foreign keys cannot cross shard boundaries.** The database on Shard 1 has no way to verify that a foreign key reference points to a valid row on Shard 2.

```sql
-- THIS CANNOT BE ENFORCED:
-- orders.user_id on Shard 2 references users.user_id on Shard 1
-- The database engine on Shard 2 can't check Shard 1.

-- The foreign key constraint would fail:
ALTER TABLE orders ADD CONSTRAINT fk_user
    FOREIGN KEY (user_id) REFERENCES users(user_id);
-- Error: referenced table doesn't exist (it's on another server)
```

**Consequences:**
- Orphaned records: delete a user on Shard 1 and their orders on Shard 2 remain, pointing to nothing.
- Invalid references: insert an order on Shard 2 with a user_id that doesn't exist on any shard.
- All referential integrity must be enforced **in application code** — the database can't help you.

```ad-warning
title: Application-Level Integrity Is Fragile
When you move referential integrity from the database to the application, you lose the database's guarantees. Every code path that writes data must manually check references. One bug, one missed check, one race condition — and you have corrupt data. This is a significant maintenance burden and source of bugs.
```

### 7. Aggregation Queries

- Queries like `COUNT(*)`, `SUM()`, `AVG()`, `MAX()`, `MIN()` across the full dataset require querying every shard and combining results.

```sql
-- "How many total orders do we have?"
-- Must run on EVERY shard:
SELECT COUNT(*) FROM orders;
-- Shard 1: 2,341,000
-- Shard 2: 2,298,000
-- Shard 3: 2,412,000
-- Shard 4: 2,350,000
-- Application must sum: 9,401,000

-- COUNT and SUM are easy to combine. AVG is trickier:
-- You can't average the averages!
-- Shard 1 AVG: $50 (1000 orders)
-- Shard 2 AVG: $75 (500 orders)
-- Wrong: ($50 + $75) / 2 = $62.50
-- Right: (1000*$50 + 500*$75) / 1500 = $58.33
-- Must compute SUM and COUNT separately, then divide.
```

```ad-danger
title: You Cannot Average Averages
When aggregating `AVG` across shards, you must collect `SUM` and `COUNT` from each shard, then compute `total_sum / total_count`. Simply averaging the per-shard averages gives an incorrect result because shards have different row counts. This applies to any weighted aggregate: standard deviation, percentiles, etc.
```

**Mitigation strategies:**
- **Pre-computed aggregates.** Maintain summary tables that each shard updates on write. The aggregation query reads the summary instead of scanning all data.
- **Data warehouse.** ETL all shard data into a single analytics database (Snowflake, BigQuery, ClickHouse) for aggregate queries.
- **CQRS pattern.** Separate the read model (optimized for queries, possibly denormalized, possibly on a single server) from the write model (sharded for throughput).

```ad-summary
title: Section Summary
- **Cross-shard queries** require scatter-gather across all shards, limited by the slowest shard.
- **Cross-shard JOINs** are impossible at the database level — co-locate related data or join in application code.
- **Resharding** is the hardest operational challenge — changing N in `MOD N` moves most of the data. Consistent hashing reduces this to ~1/N.
- **Distributed transactions** require 2PC (slow, blocking) or sagas (eventually consistent). Avoid by co-locating related data.
- **Auto-increment IDs** collide across shards — use Snowflake IDs, UUIDs, or offset-based schemes.
- **Foreign keys** cannot span shards — referential integrity must be enforced in application code.
- **Aggregation** across shards requires querying all shards and careful combination (you cannot average averages).
```

---

## Routing — How the App Finds the Right Shard

When the application needs to execute a query, it must first determine *which shard* to connect to. There are three fundamental approaches.

### 1. Application-Level Routing

- The application itself contains the sharding logic: it computes `hash(key) MOD N`, looks up a range, or consults a local configuration to determine the correct shard, then connects directly.

```csharp
// C# pseudo-code: application-level shard routing
public class ShardRouter
{
    private readonly string[] _connectionStrings;
    
    public ShardRouter(string[] connectionStrings)
    {
        _connectionStrings = connectionStrings;
    }
    
    public string GetConnectionString(int userId)
    {
        // Hash-based routing
        int shardIndex = Math.Abs(userId.GetHashCode()) % _connectionStrings.Length;
        return _connectionStrings[shardIndex];
    }
}

// Usage:
var router = new ShardRouter(new[]
{
    "Server=shard1.db.internal;Database=app;...",
    "Server=shard2.db.internal;Database=app;...",
    "Server=shard3.db.internal;Database=app;...",
    "Server=shard4.db.internal;Database=app;..."
});

string connStr = router.GetConnectionString(userId: 42);
using var conn = new MySqlConnection(connStr);
// Execute query on the correct shard
```

**Advantages:**
- No additional infrastructure — routing logic lives in the app
- Fastest routing: no extra network hop
- Full control over routing strategy

**Disadvantages:**
- Routing logic duplicated in every service that accesses the database
- Changing the routing strategy (e.g., adding a shard) requires redeploying all services
- Every developer must understand the sharding scheme — mistakes route queries to the wrong shard

### 2. Proxy Layer

- A database proxy sits between the application and the shards. The application sends queries to the proxy as if it were a normal database, and the proxy parses the query, extracts the shard key, and routes to the correct shard.

```
Application → ProxySQL / Vitess → Shard 1
                               → Shard 2
                               → Shard 3
```

**Tools:**
- **ProxySQL** — MySQL-aware proxy with query routing, connection pooling, read/write splitting
- **Vitess** — YouTube/Google's MySQL sharding middleware. Handles routing, resharding, schema changes across shards
- **ShardingSphere** — Apache project for database sharding (supports MySQL, PostgreSQL)

**Advantages:**
- Application code is simple — just connects to one endpoint
- Routing logic centralized — change it once, all apps benefit
- Can handle scatter-gather, cross-shard queries automatically
- Connection pooling across shards

**Disadvantages:**
- Additional network hop (adds latency)
- Proxy is a single point of failure (must be highly available)
- Proxy must understand SQL to parse and route queries — may not support all SQL features
- Adds operational complexity (another service to monitor, upgrade, debug)

### 3. Directory / Lookup Service

- A dedicated service maps shard key values to shards. The application asks "where is user 42?", gets back a shard address, and connects directly.

```
Application → "Where is user 42?" → Directory Service
           ← "shard-02.db.internal" ←
Application → query → shard-02.db.internal
```

**Advantages:**
- Fine-grained control — can move individual keys between shards
- Flexible rebalancing without changing routing logic
- Works with any routing strategy (range, hash, custom)

**Disadvantages:**
- Extra round trip for every query (unless cached)
- Directory is a single point of failure
- Cache invalidation when keys move between shards

### Routing Approach Comparison

| Aspect | Application-Level | Proxy Layer | Directory Service |
|---|---|---|---|
| **Extra network hop** | None | One (to proxy) | One (to directory) |
| **Single point of failure** | None | Proxy | Directory |
| **Routing flexibility** | Hardcoded in app | Centralized in proxy | Maximum (per-key) |
| **Deployment impact** | Must redeploy apps | Change proxy config | Update directory entries |
| **Complexity** | In every app | In proxy infrastructure | In directory service |
| **Cross-shard queries** | App must implement | Proxy handles automatically | App must implement |
| **SQL awareness** | App knows its queries | Proxy must parse SQL | App knows its queries |
| **Best for** | Small teams, simple sharding | Large teams, complex routing | Systems needing per-key control |

```ad-tip
title: Hybrid Approaches Are Common
In practice, many systems combine approaches. For example: use a directory for the shard mapping, cache it in the application, and fall back to the directory on cache miss. Or use a proxy for scatter-gather queries but direct connections for single-shard queries. Don't feel locked into one approach.
```

```ad-summary
title: Section Summary
- **Application-level routing** is the simplest (no extra infrastructure) but embeds routing logic in every service.
- **Proxy layers** (ProxySQL, Vitess) centralize routing and can handle cross-shard queries transparently, at the cost of an extra hop and operational complexity.
- **Directory services** offer maximum flexibility (per-key routing) but introduce a single point of failure and require aggressive caching.
- Hybrid approaches are common and often the best practical solution.
```

---

## Sharding vs Partitioning vs Clustering vs Replication

This is a common source of confusion. Here is a detailed comparison of all four techniques:

| Aspect | Partitioning | Replication | Clustering | Sharding |
|---|---|---|---|---|
| **What it is** | Split one table into pieces on the *same* server | Copy all data to multiple servers | Multiple servers share/sync the same data | Split data across multiple servers, each with *different* rows |
| **Where data lives** | One server, one database | Multiple servers, each has full copy | Multiple nodes, all have full copy (or shared disk) | Multiple servers, each has a unique subset |
| **What it scales** | Query performance on large tables | Read throughput (read from any replica) | Reads + availability | Read *and* write throughput |
| **Write scaling** | No (single server) | No (writes go to primary only) | Limited (multi-primary adds some) | Yes (writes distributed across shards) |
| **Read scaling** | Some (prune irrelevant partitions) | Yes (read from replicas) | Yes (read from any node) | Yes (smaller tables per shard) |
| **Application changes** | None (transparent to app) | Minimal (separate read/write connections) | Minimal | **Major** (routing logic, no cross-shard JOINs, distributed TX) |
| **JOINs** | Normal (same server) | Normal (same data on each replica) | Normal | Broken across shards |
| **Transactions** | Normal (same server) | Normal (on primary) | Supported (with overhead) | Distributed TX required across shards |
| **Complexity** | Low | Low-Medium | Medium | **Very High** |
| **When to use** | Large tables, improving partition pruning | Read-heavy workloads, high availability | High availability, moderate read scaling | When nothing else can handle the data volume or write throughput |
| **Examples** | MySQL `PARTITION BY RANGE/HASH` | MySQL replication, PostgreSQL streaming replication | MySQL NDB Cluster, Galera Cluster | Vitess, Citus, custom app-level sharding |

```ad-note
You can (and often should) **combine** these techniques:
- Shard your data across 4 servers, then replicate each shard for high availability.
- Partition tables within each shard to improve query performance further.
- The progression is typically: partition → replicate → cluster → shard.
```

```ad-summary
title: Section Summary
- **Partitioning** is single-server, transparent, and solves large-table performance.
- **Replication** scales reads and adds availability but doesn't help with write throughput.
- **Clustering** adds availability and some read scaling with multiple nodes sharing data.
- **Sharding** is the only technique that scales writes by distributing different data to different servers, but at the highest complexity cost.
- These techniques are not mutually exclusive — combine them for maximum effect.
```

---

## When to Shard — The Progression

```ad-important
title: Sharding Is a Last Resort
Sharding adds enormous complexity to every aspect of your system: application code, operations, deployment, debugging, monitoring, backups, schema changes, and more. Before you shard, exhaust every simpler alternative. Many teams that think they need sharding actually need better indexes or a read replica.
```

Here is the order you should try scaling approaches, from simplest to most complex:

### Step 1: Optimize Queries and Add Indexes

- Check your slow query log. Profile your queries. Add covering indexes.
- This alone fixes 90% of performance problems.
- Cost: hours of work. Complexity: zero.

```sql
-- Check slow query log (MySQL):
SET GLOBAL slow_query_log = 1;
SET GLOBAL long_query_time = 1; -- log queries taking > 1 second

-- Then examine and add indexes:
EXPLAIN SELECT * FROM orders WHERE customer_id = 42;
-- If you see "type: ALL" (full table scan), add an index:
CREATE INDEX idx_orders_customer ON orders(customer_id);
```

### Step 2: Table Partitioning

- Split large tables into partitions on the *same* server. The database handles routing automatically.
- Good for time-series data, archival, and improving partition pruning.
- Cost: hours to days. Complexity: low. See [[Table Partitioning]].

### Step 3: Read Replicas

- Add replica servers that receive copies of all writes. Route read queries to replicas, writes to the primary.
- Scales read throughput linearly (add more replicas = handle more reads).
- Cost: days. Complexity: low-medium (read/write splitting in app).

```sql
-- MySQL: Setting up a replica (simplified)
-- On primary:
CREATE USER 'replicator'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'%';
SHOW MASTER STATUS; -- note the log file and position

-- On replica:
CHANGE MASTER TO
    MASTER_HOST='primary.db.internal',
    MASTER_USER='replicator',
    MASTER_PASSWORD='password',
    MASTER_LOG_FILE='mysql-bin.000001',
    MASTER_LOG_POS=154;
START SLAVE;
```

### Step 4: Vertical Scaling

- Get a bigger server: more CPU, more RAM, faster disks (NVMe SSDs), more storage.
- Often the cheapest and fastest option if you haven't maxed out hardware yet.
- Cost: money (maybe a lot of money). Complexity: near-zero.

```ad-tip
title: Don't Underestimate Vertical Scaling
A modern database server with 128 cores, 1 TB RAM, and NVMe SSDs can handle *enormous* workloads. Many teams jump to sharding when a bigger machine would solve the problem for years. At current cloud prices, a very large RDS instance costs ~$10,000/month — less than the engineering cost of implementing sharding.
```

### Step 5: Caching Layer

- Add Redis or Memcached between your application and database. Cache frequently-read data.
- Dramatically reduces database load for read-heavy workloads.
- Cost: days. Complexity: medium (cache invalidation is hard).

### Step 6: Clustering

- Multiple database nodes sharing or syncing the same data (Galera Cluster, MySQL Group Replication).
- Adds availability and some read/write distribution.
- Cost: weeks. Complexity: medium.

### Step 7: Sharding

- Only when you have exhausted *all* of the above and your workload still exceeds what a single server (plus replicas, caching, and optimization) can handle.
- Typical triggers:
  - Dataset exceeds single-server storage (many terabytes)
  - Write throughput exceeds what one primary can handle
  - You need geo-distributed writes (different regions writing to local shards)

```ad-warning
title: The Complexity Tax
When you shard, *every* operation becomes harder:
- Adding a column? Do it on N shards.
- Running a migration? Coordinate across N shards.
- Debugging a bug? Data could be on any shard.
- Monitoring? N dashboards instead of 1.
- Backups? N backup jobs instead of 1.
- Schema out of sync between shards? Good luck.

This "complexity tax" is paid on every feature, every day, forever. Make sure the benefit justifies it.
```

```ad-summary
title: Section Summary
- Exhaust simpler alternatives before sharding: indexes, partitioning, read replicas, vertical scaling, caching, clustering.
- Each step up in complexity should only happen when the simpler approach is genuinely insufficient.
- Sharding is justified only when dataset size, write throughput, or geo-distribution requirements exceed single-server capabilities.
- The complexity tax of sharding affects every aspect of operations and development permanently.
```

---

## Real-World Examples

Understanding how major companies shard helps illustrate the principles.

### Instagram — Sharded by `user_id`

- Instagram's backend shards PostgreSQL by `user_id`.
- All photos, likes, comments, and follows for a user live on the same shard.
- Queries almost always involve a specific user → single-shard operations.
- They use a combination of PostgreSQL partitioning within each shard and application-level routing.

### Discord — Sharded by `guild_id` (Server ID)

- Discord's message storage is sharded by `guild_id`.
- All messages, reactions, and metadata for a server live on the same shard.
- This makes sense because Discord queries are always within a server context — you view messages in a server's channel, not across all servers.
- Large guilds (servers) can become hotspots — Discord has additional strategies for splitting very large guilds.

### Stripe — Sharded by `merchant_id`

- Stripe shards payment data by `merchant_id`.
- All transactions, customers, and subscriptions for a merchant live together.
- API calls almost always include the merchant context (via the API key).
- Cross-merchant queries (e.g., Stripe's internal analytics) use a separate data pipeline.

### The Common Pattern

```ad-important
title: Every Major Sharded System Follows the Same Rule
**Shard by the entity your queries center around:**
- Social media → `user_id` (queries are user-centric)
- Chat/messaging → `guild_id` / `channel_id` (queries are conversation-centric)
- Payments → `merchant_id` / `account_id` (queries are merchant-centric)
- Multi-tenant SaaS → `tenant_id` (queries are tenant-centric)
- E-commerce → `customer_id` or `shop_id` (queries are customer/shop-centric)
```

### Sharding Tools and Middleware

| Tool | Database | What It Does |
|---|---|---|
| **Vitess** | MySQL | Google/YouTube's sharding layer. Handles routing, resharding, schema management. The most mature MySQL sharding solution. |
| **Citus** | PostgreSQL | Distributed PostgreSQL extension. Adds sharding and distributed query execution to PostgreSQL transparently. |
| **MongoDB Sharding** | MongoDB | Built-in sharding. Config servers manage shard metadata; `mongos` routers handle query routing. |
| **ProxySQL** | MySQL/MariaDB | Query routing proxy. Can route based on query rules, but you configure the sharding logic yourself. |
| **ShardingSphere** | MySQL/PostgreSQL | Apache project. Database sharding middleware with Java client and proxy modes. |
| **PlanetScale** | MySQL (Vitess) | Managed Vitess-based MySQL platform with built-in sharding, branching, and schema management. |
| **CockroachDB** | (own engine) | Distributed SQL database with built-in automatic sharding. No separate middleware needed. |
| **TiDB** | MySQL-compatible | Distributed SQL that auto-shards. Compatible with MySQL protocol. |

```ad-tip
title: Managed Sharding vs DIY
Unless you have a team of experienced distributed systems engineers, prefer a managed sharding solution (Vitess, Citus, PlanetScale, CockroachDB) over building your own. The operational complexity of maintaining a custom sharding layer is enormous — let battle-tested infrastructure handle the hard parts.
```

```ad-summary
title: Section Summary
- Major companies shard by the entity their queries center around: Instagram → user_id, Discord → guild_id, Stripe → merchant_id.
- The pattern is universal: identify the primary query dimension and shard by it.
- Mature tools exist for MySQL (Vitess, ProxySQL), PostgreSQL (Citus), and MongoDB (built-in). Managed solutions (PlanetScale, CockroachDB, TiDB) reduce operational burden significantly.
```

---

## C# and Application-Level Considerations

As a C#/.NET developer, here is what changes when your application talks to a sharded database.

### Multiple Connection Strings

- Instead of one connection string, your app needs one per shard. Configuration might look like:

```json
// appsettings.json
{
  "ShardConnections": {
    "Shard0": "Server=shard0.db.internal;Database=app;User=app;Password=...",
    "Shard1": "Server=shard1.db.internal;Database=app;User=app;Password=...",
    "Shard2": "Server=shard2.db.internal;Database=app;User=app;Password=...",
    "Shard3": "Server=shard3.db.internal;Database=app;User=app;Password=..."
  }
}
```

### Shard Router Implementation

```csharp
// A simple shard router for hash-based sharding
public interface IShardRouter
{
    string GetConnectionString(int shardKey);
    IReadOnlyList<string> GetAllConnectionStrings(); // for scatter-gather
}

public class HashShardRouter : IShardRouter
{
    private readonly string[] _connectionStrings;
    
    public HashShardRouter(IConfiguration config)
    {
        var section = config.GetSection("ShardConnections");
        _connectionStrings = section.GetChildren()
            .OrderBy(c => c.Key) // Shard0, Shard1, Shard2...
            .Select(c => c.Value!)
            .ToArray();
    }
    
    public string GetConnectionString(int shardKey)
    {
        // Stable hash — don't use object.GetHashCode() 
        // (it's randomized per process in .NET)
        int hash = StableHash(shardKey);
        int shardIndex = Math.Abs(hash) % _connectionStrings.Length;
        return _connectionStrings[shardIndex];
    }
    
    public IReadOnlyList<string> GetAllConnectionStrings()
        => _connectionStrings;
    
    private static int StableHash(int value)
    {
        // Simple but stable hash — same input always gives same output
        // across processes and restarts
        unchecked
        {
            int hash = value * 2654435761; // Knuth multiplicative hash
            return hash;
        }
    }
}
```

```ad-danger
title: Do NOT Use object.GetHashCode() for Shard Routing
In .NET, `string.GetHashCode()` and `object.GetHashCode()` are **randomized per process** (hash randomization was added for security). The same string can produce different hash values in different processes or after a restart. Your shard routing will break. Always use a **deterministic hash function** like MurmurHash, xxHash, FNV, or a simple multiplicative hash.
```

### Repository Pattern with Sharding

```csharp
public class ShardedUserRepository
{
    private readonly IShardRouter _router;
    
    public ShardedUserRepository(IShardRouter router)
    {
        _router = router;
    }
    
    // Single-shard query — fast
    public async Task<User?> GetByIdAsync(int userId)
    {
        string connStr = _router.GetConnectionString(userId);
        using var conn = new MySqlConnection(connStr);
        return await conn.QuerySingleOrDefaultAsync<User>(
            "SELECT * FROM users WHERE user_id = @Id",
            new { Id = userId });
    }
    
    // Scatter-gather — queries ALL shards
    public async Task<int> GetTotalUserCountAsync()
    {
        var tasks = _router.GetAllConnectionStrings()
            .Select(async connStr =>
            {
                using var conn = new MySqlConnection(connStr);
                return await conn.ExecuteScalarAsync<int>(
                    "SELECT COUNT(*) FROM users");
            });
        
        int[] counts = await Task.WhenAll(tasks);
        return counts.Sum();
    }
    
    // Scatter-gather with merge — search across all shards
    public async Task<List<User>> SearchByNameAsync(string name)
    {
        var tasks = _router.GetAllConnectionStrings()
            .Select(async connStr =>
            {
                using var conn = new MySqlConnection(connStr);
                return (await conn.QueryAsync<User>(
                    "SELECT * FROM users WHERE name LIKE @Name",
                    new { Name = $"%{name}%" })).ToList();
            });
        
        var results = await Task.WhenAll(tasks);
        return results.SelectMany(r => r).ToList(); // merge results
    }
}
```

### EF Core Limitations

```ad-warning
title: EF Core Does Not Natively Support Sharding
Entity Framework Core has no built-in sharding support. It expects one `DbContext` to talk to one database. Workarounds:
- **Multiple DbContexts**: Create one `DbContext` per shard, each with its own connection string. The routing layer decides which `DbContext` to use.
- **DbContext factory**: A factory method that accepts a shard key and returns the correct `DbContext`.
- **Abandon EF for sharded queries**: Use Dapper (micro-ORM) for sharded access and keep EF for single-shard operations.
```

```csharp
// DbContext per shard approach
public class ShardedDbContextFactory
{
    private readonly IShardRouter _router;
    
    public ShardedDbContextFactory(IShardRouter router)
    {
        _router = router;
    }
    
    public AppDbContext CreateForUser(int userId)
    {
        string connStr = _router.GetConnectionString(userId);
        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseMySql(connStr, ServerVersion.AutoDetect(connStr))
            .Options;
        return new AppDbContext(options);
    }
}

// Usage:
using var db = _factory.CreateForUser(userId: 42);
var user = await db.Users.FindAsync(42);
```

```ad-tip
title: Dapper + Sharding Is a Natural Fit
Dapper is a micro-ORM that works directly with `IDbConnection`. Since shard routing is fundamentally about choosing the right connection, Dapper's connection-centric model fits sharding perfectly. You compute the shard, create a connection to it, and execute your query — no ORM abstraction fighting you.
```

```ad-summary
title: Section Summary
- C# apps need multiple connection strings and a routing layer to determine which shard to query.
- Always use a **deterministic hash function** — never `object.GetHashCode()` (it's randomized per process in .NET).
- Scatter-gather queries use `Task.WhenAll` to query all shards in parallel and merge results.
- EF Core doesn't support sharding natively — use multiple DbContexts or Dapper.
- Dapper's connection-centric model is a natural fit for sharded architectures.
```

---

## Limitations and Trade-offs Summary

Everything you lose (or that becomes harder) when you introduce sharding:

| Capability | Before Sharding | After Sharding |
|---|---|---|
| **JOINs** | Normal SQL JOINs | Cross-shard JOINs impossible; must join in application code or co-locate data |
| **Transactions** | ACID guaranteed by one engine | Cross-shard requires 2PC or sagas; eventual consistency |
| **Foreign keys** | Database-enforced | Must enforce in application code; no cross-shard FK |
| **Auto-increment IDs** | Simple, guaranteed unique | Collisions across shards; need Snowflake/UUID/offset schemes |
| **Aggregation (COUNT, SUM, AVG)** | One query | Scatter-gather across all shards; careful merge logic (can't average averages) |
| **Schema changes** | `ALTER TABLE` once | Must run on N shards; risk of drift if one fails |
| **Backups** | One backup job | N backup jobs; point-in-time consistency across shards is complex |
| **Monitoring** | One database to watch | N databases to monitor, N sets of metrics |
| **Debugging** | Data is in one place | "Which shard is this user on?" becomes a constant question |
| **Application complexity** | Connect to one database | Routing logic, scatter-gather, distributed error handling |
| **ORM support** | Full | Limited or none; ORMs expect one database |
| **Ad-hoc queries** | Run against one DB | Must run against every shard and merge manually |
| **Ordering (ORDER BY)** | Database handles it | Must sort merged results in application code |
| **UNIQUE constraints** | Database-enforced | Only enforced within a shard; cross-shard uniqueness needs external coordination |
| **Testing** | One database in test | Multi-shard test setup; integration tests need multiple databases |

```ad-danger
title: The Bottom Line
Sharding trades database-level guarantees (JOINs, transactions, referential integrity, unique constraints) for horizontal scalability. Every guarantee you lose must be reimplemented in application code — which is slower, more bug-prone, and harder to maintain. This trade-off is worth it only when you genuinely cannot scale any other way.
```

```ad-summary
title: Section Summary
- Sharding sacrifices nearly every convenience of a single relational database: JOINs, transactions, foreign keys, auto-increment, aggregation, schema management, and more.
- Each lost capability must be reimplemented in application code at greater cost and lower reliability.
- The complexity is permanent and compounds with every feature and operational task.
- Only shard when the scalability benefit outweighs this comprehensive cost.
```

---

## Comprehensive Summary

```ad-tip
title: Complete Summary
**Database sharding** splits a dataset across multiple independent database servers, each holding a different subset of the data. It is the most powerful horizontal scaling technique — and the most complex.

**Shard key selection** is the single most critical decision. Shard by the entity your queries center around (user_id, tenant_id, merchant_id). Good shard keys have high cardinality, even distribution, align with query patterns, and are immutable. Bad shard keys (timestamps, enums, auto-increment) create hotspots or unusable distributions.

**Three strategies** map keys to shards: *range-based* (simple, supports range queries, but hotspot-prone), *hash-based* (even distribution, but kills range queries and makes resharding expensive), and *directory-based* (maximum flexibility, but introduces a single point of failure). Consistent hashing mitigates the resharding cost of hash-based approaches.

**The hard problems** are what make sharding a last resort: cross-shard queries require scatter-gather; cross-shard JOINs are impossible at the database level; resharding means physically moving data while the system runs; distributed transactions need 2PC or sagas; auto-increment IDs collide; foreign keys can't span shards; aggregation requires careful merge logic.

**Routing** connects apps to the right shard via application-level logic (simplest), a proxy layer like Vitess (most transparent), or a directory service (most flexible). Hybrid approaches are common in practice.

**Before sharding**, exhaust simpler alternatives in this order: query optimization, table partitioning, read replicas, vertical scaling, caching, and clustering. Each step is dramatically simpler and often sufficient.

**Real-world systems** confirm the pattern: Instagram shards by user_id, Discord by guild_id, Stripe by merchant_id — always by the primary query dimension.

For **C#/.NET developers**, sharding means managing multiple connection strings, implementing a deterministic hash router (never use `object.GetHashCode()`), handling scatter-gather with `Task.WhenAll`, and accepting that EF Core has no native sharding support — Dapper is a better fit for sharded access.

Sharding permanently trades database-level guarantees (JOINs, ACID transactions, foreign keys, auto-increment, aggregation) for horizontal scalability. Every lost guarantee must be reimplemented in application code. This trade-off is justified only when dataset size, write throughput, or geo-distribution needs exceed single-server capabilities and all simpler alternatives have been exhausted.
```

---

## Related Topics

- [[Table Partitioning]] — single-server alternative to sharding; try it first
- [[Views]] — can simulate partitioned reads using `UNION ALL` across tables
- [[02 - Transactions and Concurrency|Transactions]] — distributed transaction complexity in sharded systems
- [[Real-World Database Challenges]] — mentions sharding as a scaling technique for billion-row datasets
- [[02 - Indexes|Indexes]] — optimize queries before considering sharding
- [[01 - Query Optimization|Query Optimization]] — the first line of defense before any scaling technique

---

## Further Reading

- Google Vitess documentation — the most mature MySQL sharding middleware
- Stripe's engineering blog on sharding — practical resharding lessons
- Discord's blog on message storage — sharding by guild_id at massive scale
- Martin Kleppmann, *Designing Data-Intensive Applications* — the definitive book on distributed data systems, including sharding (Chapter 6: Partitioning)
- *The Art of Scalability* by Abbott & Fisher — scalability patterns including the AKF Scale Cube, which formalizes sharding as one of three scaling dimensions
