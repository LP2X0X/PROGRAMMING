---
tags:
 - csharp
 - oop
 - keyword
---

# GetHashCode and Hash Buckets

`GetHashCode()` is a virtual method on `System.Object` that returns an `int` hash code for an object. By default, System.Object.GetHashCode() uses your object’s current location in memory to yield the hash value. Hash-based collections (`Dictionary<TKey,TValue>`, `HashSet<T>`) rely on it to achieve O(1) lookups.

---

## The Problem Hash Tables Solve

With a plain list, finding an item by key means scanning every element — O(n). Hash tables use a formula to jump **directly** to the right location, making lookups O(1).

---

## What Is a Bucket?

A hash table is internally an array of slots called **buckets**. Each bucket can hold one or more entries. The bucket is determined by:

```
Bucket index = GetHashCode() % number_of_buckets
```

### Example

Say a dictionary has 8 buckets and you store the key `"Alice"`:

```
"Alice".GetHashCode()  ->  some int, say 299
299 % 8 = 3            ->  put it in bucket 3
```

Later, when you look up `"Alice"`:

```
"Alice".GetHashCode()  ->  299 again (same input = same hash)
299 % 8 = 3            ->  go straight to bucket 3, found it
```

Buckets 0-2 and 4-7 are skipped entirely.

### Visual

```
Bucket 0:  [ ]
Bucket 1:  [ "Bob" -> 42 ]
Bucket 2:  [ ]
Bucket 3:  [ "Alice" -> 17 ]  <-- go here directly
Bucket 4:  [ "Carol" -> 88 ]
Bucket 5:  [ ]
Bucket 6:  [ ]
Bucket 7:  [ "Dave" -> 5 ]
```

---

## Collisions

Two different keys can land in the same bucket — this is called a **collision**:

```
"Alice".GetHashCode() % 8 = 3
"Eve".GetHashCode()   % 8 = 3   <-- same bucket!
```

```
Bucket 3:  [ "Alice" -> 17,  "Eve" -> 63 ]
```

When this happens, the runtime checks each entry in the bucket using `Equals()` to find the exact match.

- **`GetHashCode()`** narrows down *which bucket* (fast)
- **`Equals()`** finds the *exact match* within the bucket (slower, but only a few items)

If every key hashed to the same bucket, lookups degrade back to O(n). A good hash function distributes keys evenly across buckets to keep each bucket small.

---

## The Contract

| Rule | Why |
|---|---|
| Equal objects must produce equal hash codes | Otherwise they land in different buckets — lookup searches the wrong bucket and misses |
| Hash must be stable while in a collection | If the hash changes after insertion, the object is in one bucket but lookup computes a different one — the object is lost |
| Collisions are OK | They just mean a few extra `Equals()` checks within one bucket |

```ad-important
If `a.Equals(b)` is `true`, then `a.GetHashCode() == b.GetHashCode()` **must** also be `true`. Breaking this rule silently breaks dictionaries and hash sets.

The reverse is NOT required — two unequal objects *may* share a hash code (collision).
```

---

## Default Behavior

- **Reference types**: hash is based on object identity (memory address). Two different objects with identical field values get different hash codes.
- **Value types** (`struct`): `System.ValueType` overrides `GetHashCode()` using reflection to hash fields — slow and sometimes incorrect. Always override it for structs you use in collections.
- **String** overrides it to compute the hash from the characters in the string, not the memory address.

---

## GetHashCode Always Goes with Equals

Whenever you override `Equals()`, you must override `GetHashCode()`. The compiler warns you (`CS0659`) if you don't.

| You override | But not | What goes wrong |
|---|---|---|
| `Equals` only | `GetHashCode` | Two equal objects get different hashes -> different buckets -> dictionary/hashset can't find them |
| `GetHashCode` only | `Equals` | Same bucket, but `Equals` uses reference equality -> no match even though the data is identical |

---

## Implementation

### Modern Approach: `HashCode.Combine` (.NET Core 2.1+ / .NET 5+)

```csharp
public class Money : IEquatable<Money>
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }

    public bool Equals(Money other) =>
        other is not null && Amount == other.Amount && Currency == other.Currency;

    public override bool Equals(object obj) => Equals(obj as Money);

    public override int GetHashCode() => HashCode.Combine(Amount, Currency);
}
```

```ad-tip
Implementing `IEquatable<T>` gives you a strongly-typed `Equals(T)` that avoids the boxing overhead of `Equals(object)`. The `Equals(object)` override just delegates to it.
```

### Legacy Approach: Prime Multiplication

Before `HashCode.Combine` existed:

```csharp
public override int GetHashCode()
{
    unchecked
    {
        int hash = 17;
        hash = hash * 31 + Amount.GetHashCode();
        hash = hash * 31 + (Currency?.GetHashCode() ?? 0);
        return hash;
    }
}
```

---

## Common Pitfalls

- **Mutable fields in the hash**: if you hash a mutable field and then mutate it while the object is in a `HashSet` or used as a dictionary key, the object becomes unfindable — it's in the wrong bucket.
- **Forgetting to override**: two `Money(10, "USD")` instances won't match as dictionary keys if you only override `Equals` but not `GetHashCode`.
- **Using it for security**: `GetHashCode()` is not a cryptographic hash. Don't use it for checksums or integrity checks.

```ad-warning
Avoid including mutable fields in your `GetHashCode()` implementation if the object will be stored in a hash-based collection. Changing those fields after insertion effectively loses the object.
```

---

## Records: All of This for Free

C# 9+ `record` types auto-generate `Equals()`, `GetHashCode()`, and `IEquatable<T>` based on all properties:

```csharp
public record Money(decimal Amount, string Currency);

// Value equality and correct hashing built in — no overrides needed
```

---

## Summary

```
Dictionary.Get(key)
    1. Call key.GetHashCode()  ->  find the bucket
    2. Call key.Equals(other)  ->  find the exact match within the bucket
```

| Question | Answer |
|---|---|
| What is a bucket? | A slot in a hash table's internal array |
| How is the bucket chosen? | `GetHashCode() % bucket_count` |
| Equal objects, same hash? | **Required** |
| Same hash, equal objects? | Not required (collisions are fine) |
| Override when? | Whenever you override `Equals()` |
| Best API? | `HashCode.Combine(...)` |
| Hash mutable fields? | Avoid if object goes in a hash collection |
