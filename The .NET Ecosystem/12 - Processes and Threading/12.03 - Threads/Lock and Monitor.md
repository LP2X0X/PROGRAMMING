---
tags:
 - csharp
 - threading
 - synchronization
---

## The Problem

When two threads access the same shared data at the same time, the result is unpredictable. This is a **race condition**:

```csharp
int _balance = 1000;

// Thread A                      // Thread B
var b = _balance;   // 1000      var b = _balance;   // 1000
b -= 500;                        b -= 300;
_balance = b;       // 500       _balance = b;       // 700  ← overwrites A's write!

// Expected: 200. Actual: 700 or 500 depending on timing.
```

You need a way to say: **"only one thread at a time can execute this block of code."** That's mutual exclusion.


---

## `lock` — The Simple Way

`lock` is a C# keyword that ensures only one thread at a time enters a critical section. If a second thread hits the `lock` while another is inside, it **blocks and waits** until the first thread exits.

```csharp
private readonly object _lockObj = new();
private int _balance = 1000;

void Withdraw(int amount)
{
    lock (_lockObj)
    {
        // Only one thread at a time can be in here
        if (_balance >= amount)
        {
            _balance -= amount;
        }
    }
    // Lock is automatically released here — even if an exception is thrown
}
```

### Rules for the Lock Object

- Must be a **reference type** (not `int`, `bool`, `struct`)
- Should be **private** and **readonly** — never lock on `this`, `typeof(MyClass)`, or a string
- Should be a **dedicated object** whose only purpose is being a lock

```csharp
// GOOD
private readonly object _lock = new();

// BAD — anyone with a reference to this object can deadlock you
lock (this) { }

// BAD — string interning means unrelated code could share the same lock
lock ("myLock") { }

// BAD — Type objects are global, any code can lock on them
lock (typeof(MyClass)) { }
```

```ad-warning
`lock` on `this` or a public object means **any external code** that has a reference to your object can also lock on it — potentially causing unexpected deadlocks. Always use a private, dedicated lock object.
```


---

## `Monitor` — What `lock` Actually Does

`lock` is syntactic sugar for `Monitor.Enter` and `Monitor.Exit`. The compiler transforms your `lock` block into:

```csharp
// What you write:
lock (_lockObj)
{
    DoWork();
}

// What the compiler generates:
bool lockTaken = false;
try
{
    Monitor.Enter(_lockObj, ref lockTaken);
    DoWork();
}
finally
{
    if (lockTaken)
        Monitor.Exit(_lockObj);
}
```

The `finally` block guarantees the lock is released even if an exception is thrown.

### When to Use Monitor Directly

`Monitor` has features that `lock` doesn't expose:

| Feature | `lock` | `Monitor` |
|---|---|---|
| Basic mutual exclusion | Yes | Yes |
| Automatic `finally` release | Yes | Manual |
| `TryEnter` with timeout | No | Yes |
| `Wait` / `Pulse` signaling | No | Yes |

### TryEnter — Non-Blocking Lock Attempt

```csharp
bool lockTaken = false;
try
{
    // Try to acquire the lock, but give up after 1 second
    Monitor.TryEnter(_lockObj, TimeSpan.FromSeconds(1), ref lockTaken);

    if (lockTaken)
    {
        // Got the lock — do work
        DoWork();
    }
    else
    {
        // Couldn't get the lock in time
        Console.WriteLine("Lock was busy, skipping");
    }
}
finally
{
    if (lockTaken)
        Monitor.Exit(_lockObj);
}
```

### Wait and Pulse — Thread Signaling

`Monitor.Wait` releases the lock and puts the thread to sleep. `Monitor.Pulse` wakes up one waiting thread. This is a simple producer-consumer pattern:

```csharp
private readonly object _lock = new();
private Queue<string> _queue = new();

// Producer
void Enqueue(string item)
{
    lock (_lock)
    {
        _queue.Enqueue(item);
        Monitor.Pulse(_lock); // wake up one waiting consumer
    }
}

// Consumer
string Dequeue()
{
    lock (_lock)
    {
        while (_queue.Count == 0)
        {
            Monitor.Wait(_lock); // releases lock, sleeps, re-acquires lock on wake
        }
        return _queue.Dequeue();
    }
}
```

```ad-note
`Monitor.Wait` **releases the lock** while sleeping and **re-acquires it** before returning. This is why it must be called inside a `lock` block. `Pulse` wakes one waiting thread; `PulseAll` wakes all of them.
```


---

## How `lock` Works Under the Hood

For a detailed walkthrough of the object header → Sync Block Table → thread ID mechanism, see [[Sync Block Table and Lock Internals]].

### The Sync Block — Why You Lock on an Object

Every .NET reference type object has a hidden **object header** in memory, before the fields you actually use:

```
┌──────────────────┬──────────────────┬──────────────────────┐
│   Object Header  │  Method Table*   │   Your Fields        │
│  (sync block     │  (type pointer)  │                      │
│   index)         │                  │                      │
└──────────────────┴──────────────────┴──────────────────────┘
         ▲
    Built into EVERY reference type object.
    Points into a global Sync Block Table
    where the actual lock state is stored.
```

When you `lock` on an object, the CLR uses this header to look up the lock state — owner thread ID, recursion count, and waiting threads. **Every reference type is born ready to be a lock.** The infrastructure is baked into the object layout.

This is why you cannot lock on a value type — no object header, no sync block index, nowhere to attach lock state. And `new object()` is the cheapest possible thing that has one.

### Step by Step — What Happens at Each Stage

**Lock is FREE (uncontended):**

```
Thread A hits Monitor.Enter(_lockObj)
  → CLR reads sync block index → empty/unowned
  → Writes Thread A's ID into the sync block
  → Thread A enters immediately (~20 nanoseconds)

  Sync Block: Owner=Thread A, Count=1, ReadyQueue=[]
```

**Lock is OWNED (contended):**

```
Thread B hits Monitor.Enter(_lockObj)
  → CLR reads sync block → "Owner: Thread A" — OCCUPIED
  → Thread B does a BRIEF spin (hoping A finishes in nanoseconds)
  → Still owned → Thread B is put to SLEEP in the ready queue

  Sync Block: Owner=Thread A, Count=1, ReadyQueue=[Thread B]
  Thread B uses ZERO CPU while sleeping.
```

**Owner releases:**

```
Thread A hits Monitor.Exit(_lockObj)
  → Count decrements to 0 → lock is truly released
  → CLR checks ready queue → Thread B is waiting
  → CLR WAKES Thread B
  → Thread B writes its ID into the sync block, enters

  Sync Block: Owner=Thread B, Count=1, ReadyQueue=[]
```

This is why `lock`/`Monitor` is called a **hybrid lock** — it tries a brief spin first (in case the lock is released in nanoseconds), then falls back to a kernel wait (sleeping) for longer contention.

### Ready Queue vs Wait Queue

Each lock has **two** separate queues:

```
┌─────────────────────────────────┐
│         SYNC BLOCK              │
│                                 │
│  Owner: Thread A                │
│                                 │
│  READY QUEUE (involuntary)      │
│  → threads blocked at Enter()   │
│  → [Thread B, Thread C]         │
│                                 │
│  WAIT QUEUE (voluntary)         │
│  → threads that called Wait()   │
│  → [Thread D, Thread E]         │
└─────────────────────────────────┘
```

- **Ready queue** — threads that tried `Monitor.Enter` but the lock was taken. They wake automatically when the lock is released.
- **Wait queue** — threads that *had* the lock but voluntarily gave it up via `Monitor.Wait()`. They only wake when another thread calls `Monitor.Pulse()`, and then they move to the **ready queue** to compete for the lock.

### Reentrancy — Same Thread, Multiple Entries

The same thread CAN enter the same lock multiple times. The CLR tracks a **recursion count**:

```csharp
void MethodA()
{
    lock (_lock)           // Count: 1
    {
        MethodB();
    }                      // Exit → Count: 0 → truly released
}

void MethodB()
{
    lock (_lock)           // Same thread owns it → Count: 2, enter immediately
    {
        // no deadlock!
    }                      // Exit → Count: 1 → still locked
}
```

Each `Monitor.Exit` decrements the count. The lock is only truly released when the count hits 0. Without reentrancy, calling one locked method from another locked method on the same object would deadlock yourself.

### Exceptions Inside a Lock

The `finally` block guarantees the lock is always released — no deadlock. But the **state** you were protecting might be left inconsistent:

```csharp
lock (_lock)
{
    UpdateBalance(account, -500);  // succeeds
    UpdateLedger(account, -500);   // THROWS!
}
// Lock released by finally, but balance changed without matching ledger entry.
// Next thread sees inconsistent state.
```

The lock prevents concurrent access. It does not give you transactions. If your critical section does multiple steps and fails partway through, you need proper exception handling to roll back partial changes.

### Performance

| Situation | Cost | What happens |
|---|---|---|
| Uncontended (no competition) | ~20ns | Compare-and-swap on the sync block, no kernel call |
| Contended (brief) | ~100-500ns | Brief spin phase succeeds |
| Contended (long) | ~1-10μs+ | Kernel transition, thread sleep/wake, context switch |

**Don't fear `lock`. Fear contention.** A lock nobody competes for is practically free (~50 million lock/unlock cycles per second). A lock 10 threads fight over is a bottleneck — not because `lock` is slow, but because serializing 10 threads is inherently sequential.


---

## Deadlocks

A deadlock happens when two threads each hold a lock and wait for the other's lock:

```csharp
// Thread A                          // Thread B
lock (_lock1)                        lock (_lock2)
{                                    {
    lock (_lock2) // waits for B         lock (_lock1) // waits for A
    { }                                  { }
}                                    }
// Both threads frozen forever.
```

**Prevention**: always acquire multiple locks in the **same order** across all threads. If every thread locks `_lock1` before `_lock2`, no deadlock can occur.


---

## When to Use `lock` vs Other Primitives

| Scenario | Use |
|---|---|
| Protect shared data within one process | `lock` / `Monitor` |
| Need a timeout on acquiring the lock | `Monitor.TryEnter` |
| Need producer-consumer signaling | `Monitor.Wait` / `Pulse` (or better: use `BlockingCollection<T>`) |
| Need locking across processes | `Mutex` — see [[Mutex]] |
| Many readers, few writers | `ReaderWriterLockSlim` — see [[ReaderWriterLockSlim]] |
