---
tags:
 - csharp
 - threading
 - synchronization
---

## What It Is

A `Mutex` (mutual exclusion) is a synchronization primitive that works like `lock` / `Monitor` — only one thread can hold it at a time. The key difference: **a Mutex can work across processes**, not just across threads within one process.

| Feature | `lock` / `Monitor` | `Mutex` |
|---|---|---|
| Scope                     | Within one process   | **Across processes** (when named)     |
| ------------------------- | -------------------- | ------------------------------------- |
| Performance               | Fast (~20ns)         | Slow (~1μs) — involves OS kernel call |
| Requires dedicated object | Yes (`object _lock`) | No — is its own object                |
| Supports timeout          | Yes (`TryEnter`)     | Yes (`WaitOne(timeout)`)              |


---

## Why It Exists

`lock` can only protect shared data between threads in the **same** process. But sometimes you need to prevent two **separate processes** from doing something at the same time — for example:

- Only one instance of your application should run at a time
- Two applications sharing a file must not write simultaneously
- An installer and the app it's installing must not conflict

A **named Mutex** is visible system-wide. Any process that creates or opens a Mutex with the same name shares the same lock.


---

## Single-Instance Application — The Classic Use Case

```csharp
const string mutexName = "Global\\MyApp_SingleInstance";

using var mutex = new Mutex(initiallyOwned: false, name: mutexName, out bool createdNew);

if (!createdNew)
{
    Console.WriteLine("Another instance is already running.");
    return;
}

Console.WriteLine("Application started — I'm the only instance.");
Console.ReadLine();

// Mutex is released when disposed (or when the process exits)
```

- `initiallyOwned: false` — don't acquire the lock on creation, just create the mutex
- `name` — the system-wide name. The `Global\` prefix makes it visible across terminal sessions
- `createdNew` — `true` if this process created the mutex, `false` if it already existed (another instance is running)


---

## Using Mutex as a Cross-Process Lock

```csharp
using var mutex = new Mutex(false, "Global\\SharedFileLock");

// Wait to acquire the lock (blocks until available)
mutex.WaitOne();
try
{
    // Only one process at a time can be here
    File.AppendAllText(@"C:\Shared\log.txt", "Writing safely\n");
}
finally
{
    mutex.ReleaseMutex();
}
```

### With a Timeout

```csharp
if (mutex.WaitOne(TimeSpan.FromSeconds(5)))
{
    try
    {
        // Got the lock
        DoSharedWork();
    }
    finally
    {
        mutex.ReleaseMutex();
    }
}
else
{
    Console.WriteLine("Could not acquire mutex in time");
}
```

```ad-warning
Always call `ReleaseMutex()` in a `finally` block. If a thread terminates while holding a mutex, the next thread that waits on it will receive an `AbandonedMutexException` — the OS detects the orphaned lock and hands it over, but warns you that the protected state might be inconsistent.
```


---

## Named vs Unnamed Mutex

| Type | Scope | Use case |
|---|---|---|
| **Named** (`new Mutex(false, "name")`) | System-wide — any process can open it by name | Cross-process locking, single-instance apps |
| **Unnamed** (`new Mutex()`) | Same process only | Rarely useful — `lock` is faster for in-process work |

```ad-note
For in-process synchronization, always prefer `lock` / `Monitor` — they're orders of magnitude faster because they don't involve OS kernel transitions. Use `Mutex` only when you need cross-process coordination.
```


---

## Mutex vs Semaphore

|                        | Mutex                                       | Semaphore              |
| ---------------------- | ------------------------------------------- | ---------------------- |
| Max concurrent holders | **1** (exclusive)                           | **N** (configurable)   |
| Ownership              | The thread that acquired it must release it | Any thread can release |
| Cross-process          | Yes (named)                                 | Yes (named)            |

See [[Semaphore and SemaphoreSlim]] for when you need to allow multiple concurrent accessors.
