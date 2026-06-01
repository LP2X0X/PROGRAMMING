---
tags:
 - csharp
 - threading
 - synchronization
---

## The Problem

A `lock` is exclusive — only one thread at a time, period. But many real-world scenarios have a pattern: **reads are frequent, writes are rare**. A cache, a configuration object, a shared lookup table — most threads just want to read. Locking them all out while one thread reads is wasteful:

```csharp
// With lock — readers block each other unnecessarily
lock (_lock)
{
    return _cache[key]; // reading is safe with multiple threads!
}
```

Ten threads reading the cache simultaneously is perfectly safe — no data can be corrupted by concurrent reads. But `lock` forces them to go one at a time anyway.


---

## The Solution — Read/Write Separation

`ReaderWriterLockSlim` allows:
- **Multiple readers** at the same time (concurrent reads are safe)
- **One writer** at a time (exclusive — blocks all readers and other writers)

```
Multiple readers                One writer
at the same time                blocks everyone
                         
Thread A → reading ──┐          Thread D → writing ──┐
Thread B → reading ──┤ OK!      Thread A → BLOCKED   │
Thread C → reading ──┘          Thread B → BLOCKED   │
                                Thread C → BLOCKED   │
```


---

## Basic Usage

```csharp
private readonly ReaderWriterLockSlim _rwLock = new();
private Dictionary<string, string> _cache = new();

string Read(string key)
{
    _rwLock.EnterReadLock();
    try
    {
        return _cache[key];
    }
    finally
    {
        _rwLock.ExitReadLock();
    }
}

void Write(string key, string value)
{
    _rwLock.EnterWriteLock();
    try
    {
        _cache[key] = value;
    }
    finally
    {
        _rwLock.ExitWriteLock();
    }
}
```

Multiple threads can call `Read()` concurrently. When any thread calls `Write()`, it waits for all active readers to finish, then enters exclusively — blocking all new readers and writers until it exits.


---

## Upgradeable Read Lock

Sometimes you need to read first, then decide whether to write. Without upgradeable locks, you'd have to release the read lock, acquire a write lock, and re-check — creating a race condition window.

`EnterUpgradeableReadLock()` solves this:

```csharp
void AddIfMissing(string key, string value)
{
    _rwLock.EnterUpgradeableReadLock();
    try
    {
        // Read phase — other readers can still run concurrently
        if (!_cache.ContainsKey(key))
        {
            _rwLock.EnterWriteLock();
            try
            {
                // Write phase — exclusive access
                _cache[key] = value;
            }
            finally
            {
                _rwLock.ExitWriteLock();
            }
        }
    }
    finally
    {
        _rwLock.ExitUpgradeableReadLock();
    }
}
```

```ad-note
Only **one thread** can hold an upgradeable lock at a time (to prevent deadlocks from two threads trying to upgrade simultaneously). But it does coexist with regular read locks — other readers aren't blocked until you actually upgrade to a write lock.
```


---

## Try-Enter with Timeout

All three lock types have timeout variants:

```csharp
if (_rwLock.TryEnterReadLock(TimeSpan.FromSeconds(1)))
{
    try { /* read */ }
    finally { _rwLock.ExitReadLock(); }
}
else
{
    Console.WriteLine("Could not get read lock in time");
}

if (_rwLock.TryEnterWriteLock(TimeSpan.FromSeconds(1)))
{
    try { /* write */ }
    finally { _rwLock.ExitWriteLock(); }
}
```


---

## When to Use It vs `lock`

| Scenario | Best choice |
|---|---|
| Reads and writes are roughly equal | `lock` — simpler and `ReaderWriterLockSlim` adds overhead |
| Reads vastly outnumber writes (90%+ reads) | `ReaderWriterLockSlim` — readers run concurrently |
| Short critical sections | `lock` — the overhead of reader/writer tracking isn't worth it |
| Long-held read operations | `ReaderWriterLockSlim` — avoids blocking other readers |

```ad-warning
`ReaderWriterLockSlim` is **not** `IDisposable`-pattern safe with `using` — you must use `try/finally` to ensure locks are released. Also, it is **not async-compatible** — there is no `EnterReadLockAsync()`. For async scenarios, consider other patterns or a custom async reader-writer lock.
```


---

## Properties for Diagnostics

```csharp
Console.WriteLine($"Active readers:   {_rwLock.CurrentReadCount}");
Console.WriteLine($"Waiting readers:  {_rwLock.WaitingReadCount}");
Console.WriteLine($"Waiting writers:  {_rwLock.WaitingWriteCount}");
Console.WriteLine($"Is write held:    {_rwLock.IsWriteLockHeld}");
```
