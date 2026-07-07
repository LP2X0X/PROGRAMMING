---
tags:
  - csharp
  - ado-net
  - connection-pooling
aliases:
  - ADO.NET Connection Pooling
  - Pool Exhaustion
---

## Connection Pooling

```ad-note
title: What You'll Learn
**Connection pooling** is one of the most important performance features in ADO.NET. Opening a database connection is expensive — TCP handshake, TLS negotiation, authentication, session setup. The pool transparently reuses connections so your application pays this cost only once. This note covers how pooling works, how to configure it, why `using` is non-negotiable, how to diagnose pool exhaustion, and advanced pooling scenarios.
```

---

## Table of Contents

- [[#Why Connection Pooling Matters]]
- [[#How the Pool Works]]
- [[#Pool Configuration]]
- [[#The using Statement is Non-Negotiable]]
- [[#Pool Partitioning — One Pool Per Connection String]]
- [[#Diagnosing Pool Exhaustion]]
- [[#Clearing the Pool]]
- [[#Pooling Across Providers]]
- [[#Advanced Scenarios]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]

---

## Why Connection Pooling Matters

Opening a physical database connection involves:

1. **DNS resolution** of the server hostname
2. **TCP handshake** (SYN → SYN-ACK → ACK)
3. **TLS/SSL negotiation** (if encryption is enabled — multiple round trips)
4. **Authentication** (sending credentials, server validation)
5. **Session initialization** (setting options, default database, transaction isolation level)

This process takes ==10–100+ milliseconds== depending on network latency, encryption, and authentication method. For an API handling 1,000 requests/second, opening a new connection per request would add 10–100 seconds of cumulative overhead per second — clearly unsustainable.

**Connection pooling** solves this by maintaining a cache of open, ready-to-use connections:

| Approach | Connection Time | Overhead at 1,000 req/s |
|---|---|---|
| No pooling (new connection each time) | ~50ms per connection | ~50 seconds/second of pure connection overhead |
| With pooling (reuse from pool) | ~0.01ms (pool lookup) | ~10 milliseconds/second |

That's a **~5,000x** reduction in connection overhead.

```ad-note
title: Section Summary
- Opening a physical connection is expensive (10-100+ ms): DNS, TCP, TLS, auth, session setup
- Connection pooling reuses connections, reducing overhead by orders of magnitude
- Pooling is enabled by default in all major ADO.NET providers
```

---

## How the Pool Works

The pool is managed transparently by the data provider. Here is the lifecycle:

### Step-by-Step Flow

```
1. conn.Open()  ─┬─► Pool has idle connection with matching conn string?
                  │     YES → return existing connection (sub-millisecond)
                  │     NO  → Pool at Max Pool Size?
                  │             NO  → create new physical connection, add to pool, return it
                  │             YES → WAIT until a connection is returned or timeout expires
                  │
2. Use connection (execute queries, read data)
                  │
3. conn.Dispose() ──► Connection returned to pool (NOT physically closed)
                      │
                      └─► Pool marks connection as idle, available for next Open()
```

### Key Behaviors

- **`Open()`** does not always create a new connection — it may retrieve one from the pool
- **`Close()` / `Dispose()`** does not physically close the connection — it ==returns it to the pool==
- The physical connection stays open in the background, ready for the next request
- If the pool is full (`Max Pool Size` reached), `Open()` blocks the calling thread until a connection becomes available or `Connection Timeout` expires
- The pool periodically prunes idle connections that exceed `Connection Lifetime`

```csharp
// What appears to happen vs what actually happens:

using var conn = new SqlConnection(connStr);
conn.Open();    // ← Looks like: "open a new connection"
                //    Actually:  "get an existing connection from the pool (or create one)"

// ... use the connection ...

// conn.Dispose() ← Looks like: "close the connection"
//                   Actually:  "return the connection to the pool for reuse"
```

```ad-info
title: Connection Validation
When a connection is retrieved from the pool, the provider checks if it's still alive. If the server has dropped the connection (due to timeout, restart, or network failure), the pool discards it and tries the next one. This is why you might see a brief delay on the first request after a server restart — the pool is cycling through stale connections.
```

```ad-note
title: Section Summary
- `Open()` retrieves from pool or creates new; `Dispose()` returns to pool (not physically closed)
- If pool is at max capacity, `Open()` blocks until a connection is returned or timeout expires
- The provider validates connections on retrieval and discards stale ones automatically
```

---

## Pool Configuration

Pool behavior is controlled via [[Connection Strings|connection string]] parameters:

### SQL Server (`Microsoft.Data.SqlClient`)

| Parameter | Default | Purpose |
|---|---|---|
| `Pooling` | `true` | Enable or disable pooling |
| `Min Pool Size` | `0` | Minimum connections maintained (even when idle) |
| `Max Pool Size` | `100` | Maximum connections allowed |
| `Connection Lifetime` | `0` (infinite) | Max seconds a connection lives before being destroyed on return |
| `Connection Idle Timeout` | `300` (5 min) | Seconds before an idle connection above `Min Pool Size` is pruned |
| `Load Balance Timeout` | `0` | Minimum time (seconds) connection lives in pool before destruction |

### MySQL / MariaDB (`MySqlConnector`)

| Parameter | Default | Purpose |
|---|---|---|
| `Pooling` | `true` | Enable or disable pooling |
| `MinimumPoolSize` | `0` | Minimum pool size |
| `MaximumPoolSize` | `100` | Maximum pool size |
| `ConnectionLifeTime` | `0` (infinite) | Max seconds a connection lives |
| `ConnectionIdleTimeout` | `180` (3 min) | Seconds before idle pruning |

### PostgreSQL (`Npgsql`)

| Parameter | Default | Purpose |
|---|---|---|
| `Pooling` | `true` | Enable or disable pooling |
| `Minimum Pool Size` | `0` | Minimum pool size |
| `Maximum Pool Size` | `100` | Maximum pool size |
| `Connection Idle Lifetime` | `300` (5 min) | Seconds before idle pruning |
| `Connection Pruning Interval` | `10` | How often (seconds) the pool checks for idle connections |

### Tuning Guidelines

```csharp
// Example: tuned connection string for a high-traffic API
"Server=localhost;Database=MyDb;Integrated Security=true;" +
"Min Pool Size=10;" +     // keep 10 warm connections ready
"Max Pool Size=200;" +    // allow up to 200 under peak load
"Connection Lifetime=300" // recycle connections every 5 minutes
```

| Scenario | Min Pool Size | Max Pool Size | Notes |
|---|---|---|---|
| Low-traffic internal tool | `0` | `20` | Small pool, no warm connections needed |
| Standard web API | `5-10` | `100` | Keep some warm, handle bursts |
| High-traffic API | `20-50` | `200-500` | More warm connections, higher ceiling |
| Background job processor | `1-5` | `20-50` | Steady load, fewer spikes |

```ad-warning
title: Common Misconception
"Setting Max Pool Size very high (like 10,000) makes my app handle more load." This is ==false and counterproductive==. The database has a maximum number of concurrent connections it can handle (typically 100-500 for SQL Server, depending on hardware). Setting the pool size above the database's capacity just means more connections fighting for resources, causing lock contention, memory pressure, and worse performance. The pool size should match your application's actual concurrency needs, not be arbitrarily high.
```

```ad-note
title: Section Summary
- Pooling is on by default with `Max Pool Size=100` and `Min Pool Size=0` for most providers
- Tune pool sizes based on application traffic patterns and database capacity
- Don't set Max Pool Size higher than the database can handle
- Use `Connection Lifetime` to recycle connections periodically (useful for load-balanced databases)
```

---

## The using Statement is Non-Negotiable

This is the ==single most important rule in ADO.NET==: **always wrap connections in a `using` statement** (or `using` declaration). If you don't, the connection is never returned to the pool, and eventually the pool fills up.

### Correct Patterns

```csharp
// ✅ Pattern 1: using declaration (C# 8+, preferred)
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();
// ... use connection ...
// Dispose() called automatically at end of scope → returned to pool

// ✅ Pattern 2: using statement (explicit scope)
using (var conn = new SqlConnection(connStr))
{
    await conn.OpenAsync();
    // ... use connection ...
} // Dispose() called here → returned to pool

// ✅ Pattern 3: try/finally (rare, but valid)
SqlConnection? conn = null;
try
{
    conn = new SqlConnection(connStr);
    await conn.OpenAsync();
    // ... use connection ...
}
finally
{
    conn?.Dispose(); // returned to pool
}
```

### The Leak Scenario

```csharp
// ❌ CATASTROPHIC — connection leaked
public async Task<User> GetUserAsync(int id)
{
    var conn = new SqlConnection(connStr);
    await conn.OpenAsync();

    var cmd = new SqlCommand("SELECT Name FROM Users WHERE Id = @id", conn);
    cmd.Parameters.AddWithValue("@id", id);

    var name = (string?)await cmd.ExecuteScalarAsync();
    return new User { Id = id, Name = name };
    // conn is never disposed!
    // Connection is NOT returned to the pool
    // After 100 calls (default Max Pool Size), app hangs permanently
}
```

What happens step by step:

1. Each call creates a connection and never disposes it
2. After 100 calls (default `Max Pool Size`), all pool slots are occupied by leaked connections
3. The 101st call to `Open()` blocks, waiting for a connection to be returned
4. No connection will ever be returned (they're all leaked)
5. After `Connection Timeout` seconds (default 15), an exception: ==`System.InvalidOperationException: Timeout expired. The timeout period elapsed prior to obtaining a connection from the pool.`==
6. The application is now **permanently broken** until restarted

```ad-warning
title: Even Exception Paths Must Dispose
A common mistake is disposing the connection in the "happy path" but not when an exception occurs:

```csharp
// ❌ LEAKS on exception
var conn = new SqlConnection(connStr);
conn.Open();
var result = DoSomethingThatMightThrow(conn);  // if this throws...
conn.Close();  // ...this line never executes → leaked!

// ✅ using handles both paths
using var conn = new SqlConnection(connStr);
conn.Open();
var result = DoSomethingThatMightThrow(conn);  // even if this throws...
// Dispose() runs in the finally block → connection returned to pool
```

The `using` statement compiles to a `try/finally` block, so `Dispose()` is called regardless of whether an exception occurs. This is not optional — it's a correctness requirement.
```

```ad-note
title: Section Summary
- **Always** use `using` with `DbConnection` — this is the #1 rule of ADO.NET
- Not disposing connections causes pool exhaustion, which permanently hangs the application
- `Dispose()` returns the connection to the pool (doesn't physically close it)
- `using` works even when exceptions occur — it compiles to `try/finally`
```

---

## Pool Partitioning — One Pool Per Connection String

The connection pool is ==partitioned by exact connection string match==. Two connections with different connection strings (even slightly different) go to different pools.

```csharp
// These create THREE separate pools, each with its own Max Pool Size:

var conn1 = new SqlConnection("Server=localhost;Database=MyDb;Integrated Security=true");
// Pool 1: "Server=localhost;Database=MyDb;Integrated Security=true"

var conn2 = new SqlConnection("Server=localhost;Database=OtherDb;Integrated Security=true");
// Pool 2: different Database → different pool

var conn3 = new SqlConnection("Server=LOCALHOST;Database=MyDb;Integrated Security=true");
// Pool 3: "LOCALHOST" vs "localhost" → DIFFERENT pool (string comparison is exact!)
```

```ad-warning
title: Case Sensitivity Trap
Connection string matching is ==exact string comparison==, not semantic comparison. `Server=localhost` and `Server=LOCALHOST` create separate pools, even though they resolve to the same server. Similarly, `Integrated Security=true` and `Integrated Security=True` may create separate pools depending on the provider. Always use a ==single, centralized connection string== rather than constructing it in multiple places.
```

### Implications

- **Use a single connection string source** — read from configuration once, share the same string everywhere
- **Avoid dynamically building connection strings** per request — each unique string creates a new pool
- **Per-user connection strings** (e.g., different credentials per user) each get their own pool, which can lead to pool proliferation
- **Total connections** across all pools can exceed what you'd expect if you have many distinct strings

```ad-note
title: Section Summary
- Pools are partitioned by exact connection string match (case-sensitive)
- Different strings = different pools, even if they connect to the same server
- Use a centralized, consistent connection string to avoid pool proliferation
```

---

## Diagnosing Pool Exhaustion

Pool exhaustion is the most common ADO.NET performance issue. Here's how to detect and fix it.

### Symptoms

- `System.InvalidOperationException: Timeout expired. The timeout period elapsed prior to obtaining a connection from the pool.`
- Application gradually slows down and then hangs
- Database shows fewer connections than expected (the connections are "owned" by the pool but not being used or returned)
- Restarting the application temporarily fixes the problem

### Diagnostic Tools

#### SQL Server — Check Active Connections

```sql
-- See all connections from your application
SELECT 
    s.session_id,
    s.login_name,
    s.host_name,
    s.program_name,        -- matches Application Name in connection string
    s.status,
    s.last_request_start_time,
    s.last_request_end_time
FROM sys.dm_exec_sessions s
WHERE s.program_name LIKE '%MyApp%'
ORDER BY s.last_request_start_time DESC;

-- Count connections per application
SELECT program_name, COUNT(*) AS connection_count
FROM sys.dm_exec_sessions
GROUP BY program_name
ORDER BY connection_count DESC;
```

#### .NET Performance Counters

```csharp
// Add event listener for SqlClient connection pool events
// Available in Microsoft.Data.SqlClient 4.0+

// In appsettings.json, enable event counters:
// Then use dotnet-counters:
// dotnet-counters monitor --process-id <pid> Microsoft.Data.SqlClient.EventSource
```

#### Code-Level Detection

```csharp
// Quick diagnostic: log pool statistics (SQL Server specific)
using var conn = new SqlConnection(connStr);
await conn.OpenAsync();

// After opening, check pool stats via connection string builder
var builder = new SqlConnectionStringBuilder(conn.ConnectionString);
Console.WriteLine($"Max Pool Size: {builder.MaxPoolSize}");

// More practical: wrap connection creation with timeout logging
public static async Task<SqlConnection> GetConnectionAsync(string connStr)
{
    var sw = Stopwatch.StartNew();
    var conn = new SqlConnection(connStr);
    await conn.OpenAsync();
    sw.Stop();

    if (sw.ElapsedMilliseconds > 1000) // pool wait > 1 second = warning sign
    {
        Log.Warning("Connection pool delay: {Elapsed}ms — possible pool exhaustion", 
            sw.ElapsedMilliseconds);
    }
    return conn;
}
```

### Root Cause Checklist

1. **Missing `using` statements** — the #1 cause. Search your codebase for `new SqlConnection` without `using`
2. **Long-running transactions** holding connections open
3. **Blocking code** on the connection (e.g., `Thread.Sleep` while holding a connection)
4. **`Max Pool Size` too low** for actual concurrency
5. **Connection string proliferation** — many pools, each small
6. **Connection leaks in error paths** — exceptions bypass `Close()` calls

```ad-note
title: Section Summary
- Pool exhaustion manifests as timeout exceptions and application hangs
- Use SQL Server DMVs, .NET event counters, and timing diagnostics to detect issues
- The root cause is almost always missing `using` statements or long-held connections
```

---

## Clearing the Pool

Rarely, you need to manually clear the connection pool — typically after a database server restart, failover, or network disruption where all pooled connections have gone stale.

```csharp
// Clear ALL pools (all connection strings)
SqlConnection.ClearAllPools();

// Clear the pool for a specific connection string
using var conn = new SqlConnection(connStr);
SqlConnection.ClearPool(conn);    // clears the pool associated with this connection string
```

### When to Clear

| Scenario | Action |
|---|---|
| Database server restarted | `ClearAllPools()` — all cached connections are stale |
| Failover to a replica | `ClearAllPools()` — connections point to old primary |
| Network disruption resolved | Consider clearing if you see many stale connection errors |
| Password rotation | `ClearAllPools()` — cached connections use old credentials |
| Normal operation | Never clear — the pool handles lifecycle automatically |

```ad-warning
title: Don't Clear Pools Routinely
Clearing pools is an ==emergency measure==, not a routine operation. It forces all connections to be recreated from scratch, temporarily eliminating the performance benefit of pooling. The pool already handles stale connection detection automatically. Only clear when you know all cached connections are invalid.
```

### Provider-Specific Clear Methods

```csharp
// SQL Server
SqlConnection.ClearAllPools();
SqlConnection.ClearPool(conn);

// MySQL (MySqlConnector)
MySqlConnection.ClearAllPools();
MySqlConnection.ClearPool(conn);

// PostgreSQL (Npgsql)
NpgsqlConnection.ClearAllPools();
NpgsqlConnection.ClearPool(conn);
```

```ad-note
title: Section Summary
- Use `ClearAllPools()` after server restarts, failovers, or credential rotation
- Don't clear pools routinely — it's an emergency measure
- All major providers support `ClearAllPools()` and `ClearPool(conn)`
```

---

## Pooling Across Providers

All major ADO.NET providers implement connection pooling, but with slightly different behaviors:

| Feature | SQL Server | MySQL (MySqlConnector) | PostgreSQL (Npgsql) |
|---|---|---|---|
| Pooling on by default | Yes | Yes | Yes |
| Default max pool size | 100 | 100 | 100 |
| Pool per connection string | Yes | Yes | Yes |
| Pool per process | Yes | Yes | Yes |
| Automatic stale detection | Yes | Yes (ping before reuse) | Yes |
| Clear pool API | `ClearAllPools()` | `ClearAllPools()` | `ClearAllPools()` |
| Async open from pool | Yes | Yes | Yes |

```ad-info
title: SQLite is Different
SQLite is an ==in-process== database — there's no network connection, no TCP, no authentication. Opening a SQLite connection just opens a file handle, which is extremely fast (~microseconds). Connection pooling still exists for SQLite (managed by `Microsoft.Data.Sqlite`), but the performance benefit is much smaller than with network databases. The pool mainly avoids repeated file handle allocation and WAL setup.
```

```ad-note
title: Section Summary
- All major providers implement pooling with similar defaults and APIs
- SQLite pooling exists but has minimal benefit since there's no network overhead
```

---

## Advanced Scenarios

### Per-User Connection Strings

In multi-tenant applications where each user connects with different credentials:

```csharp
// Each unique connection string creates its own pool
// With 1,000 users, you get 1,000 pools × Max Pool Size connections
// This can overwhelm the database!

// Solution: use a shared service account and implement authorization at the app level
// ONE pool, shared across all users
string sharedConnStr = "Server=localhost;Database=MyDb;User Id=app_service;Password=shared";
```

```ad-important
title: Pool Proliferation in Multi-Tenant Apps
If your application generates unique connection strings per user or tenant, each string creates a separate pool. With 500 tenants and `Max Pool Size=100`, you could potentially open 50,000 physical connections. Solutions:
1. Use a shared service account (one pool for all tenants)
2. Use `SET` statements after connecting to switch database context per tenant
3. Reduce `Max Pool Size` per connection string
4. Use PostgreSQL's `pgBouncer` or a similar external connection pooler
```

### Async Pooling Behavior

```csharp
// OpenAsync() respects the pool just like Open()
using var conn = new SqlConnection(connStr);
await conn.OpenAsync(); // Gets from pool asynchronously — does NOT block a thread while waiting

// This is critical for ASP.NET Core: synchronous Open() blocks a thread pool thread,
// while OpenAsync() releases the thread while waiting for a pool slot
```

```ad-note
title: Section Summary
- Per-user connection strings cause pool proliferation — prefer shared service accounts
- Use `OpenAsync()` in async applications to avoid blocking thread pool threads while waiting for pool slots
```

---

## Comprehensive Summary

```ad-important
title: Key Takeaways
**Connection pooling** reuses database connections instead of creating new ones each time, reducing connection overhead by ==~5,000x==. It is enabled by default in all major ADO.NET providers.

**How it works**: `Open()` retrieves from the pool (or creates new), `Dispose()` returns to the pool (not physically closed). If the pool is full, `Open()` blocks until a connection is returned or timeout expires.

**The #1 rule**: ==Always use `using` with `DbConnection`==. Forgetting to dispose a connection leaks it from the pool. After enough leaks, the pool is exhausted and the application hangs permanently with "timeout expired" errors.

**Configuration**: `Max Pool Size` (default 100), `Min Pool Size` (default 0), `Connection Lifetime` (default infinite). Tune based on actual concurrency needs and database capacity — don't set Max Pool Size higher than the database can handle.

**Pool partitioning**: Each unique connection string gets its own pool (exact string match, case-sensitive). Use a single, centralized connection string to avoid pool proliferation.

**Diagnostics**: Monitor for long `Open()` times, use SQL Server DMVs to count active connections, and search code for `new SqlConnection` without `using`. Pool exhaustion is almost always caused by missing `using` statements.
```

---

## Related Topics

- [[ADO.NET Overview]] — architecture context for where pooling fits
- [[Connection Strings]] — pool configuration parameters
- [[DbConnection]] — the connection lifecycle that pooling wraps
- [[Data Providers]] — provider-specific pooling behaviors
- [[Transactions in ADO.NET]] — how transactions interact with pooled connections
- [[IDisposable and the Dispose Pattern]] — why `using` works the way it does
