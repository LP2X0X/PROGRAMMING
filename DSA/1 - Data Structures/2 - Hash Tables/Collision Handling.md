---
tags:
  - algorithms
  - data-structure
  - hash-table
  - collision
---

## 🔹 Why Collisions Happen

A hash function maps a potentially infinite key space to a finite number of buckets. By the **pigeonhole principle**, if you have more possible keys than slots, at least two keys must map to the same index.

Even a perfect-looking hash function cannot escape this: with `m` buckets and a universe of keys `|U| >> m`, collisions are inevitable. The goal is not to eliminate collisions but to **handle them efficiently**.

```
  Keys (infinite universe)          Buckets (finite)
  ┌────────────────────┐            ┌───────┐
  │  "alice"      ─────────────────>│  [0]  │
  │  "bob"        ─────────────┐    ├───────┤
  │  "charlie"    ────────┐    └───>│  [1]  │  <-- collision!
  │  "dave"       ─────┐  └───────> │  [1]  │  <-- collision!
  │  "eve"        ──┐  └──────────> │  [2]  │
  │  ...            │               ├───────┤
  │  (billions)     └──────────────>│  [3]  │
  └────────────────────┘            └───────┘
                                    m = 4 slots
```

Key insight: even with a uniform hash function, the **birthday paradox** means collisions appear much sooner than you'd expect. With `m` buckets, the probability of a collision exceeds 50% after only about `sqrt(m)` insertions.

---

## 🔹 Chaining (Separate Chaining)

Each bucket holds a pointer to a **linked list** (or other dynamic collection) of all entries that hash to that index. Colliding keys simply get appended to the same list.

### How It Works

```
  Hash Table (m = 7 buckets)

  Index   Bucket
  ┌───┐
  │ 0 │──> NULL
  ├───┤
  │ 1 │──>┌──────────────┐    ┌──────────────┐
  │   │   │ key: "bob"   │───>│ key: "dave"  │───> NULL
  │   │   │ val: 42      │    │ val: 99      │
  │   │   └──────────────┘    └──────────────┘
  ├───┤
  │ 2 │──>┌──────────────┐
  │   │   │ key: "alice" │───> NULL
  │   │   │ val: 10      │
  │   │   └──────────────┘
  ├───┤
  │ 3 │──> NULL
  ├───┤
  │ 4 │──>┌──────────────┐    ┌──────────────┐    ┌──────────────┐
  │   │   │ key: "eve"   │───>│ key: "frank" │───>│ key: "grace" │───> NULL
  │   │   │ val: 7       │    │ val: 55      │    │ val: 31      │
  │   │   └──────────────┘    └──────────────┘    └──────────────┘
  ├───┤
  │ 5 │──> NULL
  ├───┤
  │ 6 │──>┌──────────────┐
  │   │   │ key: "charlie│───> NULL
  │   │   │ val: 88      │
  │   │   └──────────────┘
  └───┘

  n = 6 entries, m = 7 buckets, load factor alpha = 6/7 ≈ 0.86
  Bucket 1 has 2 entries (collision between "bob" and "dave")
  Bucket 4 has 3 entries (collision among "eve", "frank", "grace")
```

### Operations

**Insert** -- hash the key, go to that bucket, prepend/append to the list.

```cpp
void insert(HashTable& table, Key k, Value v) {
    int idx = hash(k) % table.size;
    // Optionally check if key already exists (update)
    table.buckets[idx].push_front({k, v});  // O(1) prepend
}
```

**Lookup** -- hash the key, go to that bucket, walk the list comparing keys.

```cpp
Value* lookup(HashTable& table, Key k) {
    int idx = hash(k) % table.size;
    for (auto& entry : table.buckets[idx]) {
        if (entry.key == k) return &entry.value;
    }
    return nullptr;  // not found
}
```

**Delete** -- hash the key, go to that bucket, find and unlink the node.

```cpp
bool remove(HashTable& table, Key k) {
    int idx = hash(k) % table.size;
    // Find node with matching key in the linked list and remove it
    return table.buckets[idx].remove(k);  // standard list removal
}
```

### Complexity

| Operation | Average    | Worst Case |
|-----------|------------|------------|
| Insert    | O(1)       | O(1)*      |
| Lookup    | O(1 + alpha) | O(n)       |
| Delete    | O(1 + alpha) | O(n)       |

*Insert is O(1) if you don't check for duplicates (just prepend). If you check for existing key first, it's O(1 + alpha) average.

`alpha` (load factor) = n/m. When alpha is small (e.g., < 1), the average chain length is short and everything is effectively O(1).

Worst case O(n) happens when all keys hash to the same bucket -- a single chain of length n.

### Pros and Cons

**Pros:**
- Simple to implement
- The table never "fills up" -- you can always add more entries
- Graceful degradation -- performance degrades linearly with load factor
- Deletion is straightforward (just unlink a node)

**Cons:**
- Extra memory for linked list pointers (each node needs a `next` pointer)
- Cache-unfriendly -- linked list nodes are scattered in memory
- With very long chains, performance approaches O(n)

---

## 🔹 Open Addressing

All entries are stored **directly in the table array** -- no linked lists, no extra pointers. When a collision occurs, you **probe** for the next available slot according to a probing strategy.

```
  Chaining:  table[i] --> node --> node --> node    (external storage)
  Open Addr: table[i], table[i+1], table[i+2], ... (in-place storage)
```

### Linear Probing

When a collision occurs at index `h`, try `h+1`, `h+2`, `h+3`, ... (wrapping around).

```
  Probe sequence: h(k), h(k)+1, h(k)+2, h(k)+3, ... (mod m)
```

**Example: Inserting keys with linear probing (m = 8)**

```
  Insert "alice" -> hash = 2        Insert "bob" -> hash = 5
  ┌───┬───┬───────┬───┬───┬─────┬───┬───┐
  │   │   │ alice │   │   │ bob │   │   │
  └───┴───┴───────┴───┴───┴─────┴───┴───┘
    0   1     2     3   4    5    6   7

  Insert "charlie" -> hash = 2 (collision!)
  Probe: index 2 (occupied) -> try 3 (empty!) -> place at 3
  ┌───┬───┬───────┬─────────┬───┬─────┬───┬───┐
  │   │   │ alice │ charlie │   │ bob │   │   │
  └───┴───┴───────┴─────────┴───┴─────┴───┴───┘
    0   1     2        3      4    5    6   7

  Insert "dave" -> hash = 2 (collision!)
  Probe: 2 (occupied) -> 3 (occupied) -> 4 (empty!) -> place at 4
  ┌───┬───┬───────┬─────────┬──────┬─────┬───┬───┐
  │   │   │ alice │ charlie │ dave │ bob │   │   │
  └───┴───┴───────┴─────────┴──────┴─────┴───┴───┘
    0   1     2        3        4     5    6   7

  Insert "eve" -> hash = 5 (collision!)
  Probe: 5 (occupied) -> 6 (empty!) -> place at 6
  ┌───┬───┬───────┬─────────┬──────┬─────┬─────┬───┐
  │   │   │ alice │ charlie │ dave │ bob │ eve │   │
  └───┴───┴───────┴─────────┴──────┴─────┴─────┴───┘
    0   1     2        3        4     5     6    7
```

**Primary Clustering Problem:**

Notice how indices 2-6 formed one continuous block. Any new key hashing to 2, 3, 4, 5, or 6 must probe past the entire cluster. Clusters grow and merge, making the problem progressively worse.

```
  Primary clustering -- big runs of occupied slots attract more entries:

  Before:  [   ] [   ] [###] [###] [###] [   ] [###] [###]
                         ^^^ cluster A ^^^       ^cluster B^

  After more inserts:
           [   ] [   ] [###] [###] [###] [###] [###] [###]
                         ^^^^^^^^^ clusters merged! ^^^^^^^
                         Any hash to 2-7 probes the whole run
```

### Quadratic Probing

Instead of stepping by 1 each time, step by increasing squares: try `h+1`, `h+4`, `h+9`, `h+16`, ...

```
  Probe sequence: h(k), h(k)+1^2, h(k)+2^2, h(k)+3^2, ... (mod m)
                  h(k), h(k)+1,   h(k)+4,   h(k)+9,   ...
```

This spreads out the probes, reducing primary clustering.

**Example: Quadratic probing (m = 11)**

```
  Key hashes to index 3, but index 3 is occupied:

  Linear probing tries:     3, 4, 5, 6, 7, ...    (consecutive)
  Quadratic probing tries:  3, 4, 7, 12%11=1, ... (spread out)

  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬────┐
  │   │ 4 │   │ 1 │ 2 │   │   │ 3 │   │   │    │
  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴────┘
    0   1   2   3   4   5   6   7   8   9   10
            ^       ^               ^
            │       │               └─ probe 3: 3 + 2^2 = 7
            │       └─ probe 2: 3 + 1^2 = 4
            └─ probe 4: 3 + 3^2 = 12 mod 11 = 1

  Numbers inside cells show the probe order
```

**Caveat:** Quadratic probing eliminates primary clustering but introduces **secondary clustering** -- keys that hash to the same index follow the same probe sequence. Also, it may not visit all slots unless `m` is prime and the table is less than half full.

### Double Hashing

Use a second, independent hash function to determine the **step size**:

```
  Probe sequence: h1(k), h1(k) + h2(k), h1(k) + 2*h2(k), ... (mod m)
```

Each key gets its own unique step size, so even keys that collide at the same index take completely different paths.

```cpp
int probe(Key k, int i, int m) {
    int h1 = hash1(k) % m;
    int h2 = 1 + (hash2(k) % (m - 1));  // h2 must never be 0
    return (h1 + i * h2) % m;
}
```

A common choice: `h2(k) = 1 + (k mod (m - 1))` where `m` is prime.

### Visual Comparison of All Three Strategies

```
  All three keys hash to index 3. Table size m = 13.

  LINEAR PROBING (step = 1 each time):
  Probe:  3 -> 4 -> 5 -> 6 -> 7 -> ...
  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬────┬────┬────┐
  │   │   │   │ X │ 1 │ 2 │ 3 │ 4 │   │   │    │    │    │
  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴────┴────┴────┘
    0   1   2   3   4   5   6   7   8   9   10   11   12
                ^^^^^^^^^^^^^^^^^^
                consecutive -- causes clustering

  QUADRATIC PROBING (step = 1, 4, 9, 16, ...):
  Probe:  3 -> 4 -> 7 -> 12 -> 8 -> ...
  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬────┬────┬────┐
  │   │   │   │ X │ 1 │   │   │ 2 │ 4 │   │    │    │  3 │
  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴────┴────┴────┘
    0   1   2   3   4   5   6   7   8   9   10   11   12
                ^   ^           ^   ^                   ^
                spread out -- less clustering

  DOUBLE HASHING (step = h2(k), e.g., h2 = 5):
  Probe:  3 -> 8 -> 0 -> 5 -> 10 -> ...
  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬────┬────┬────┐
  │ 2 │   │   │ X │   │ 3 │   │   │ 1 │   │  4 │    │    │
  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴────┴────┴────┘
    0   1   2   3   4   5   6   7   8   9   10   11   12
    ^           ^       ^           ^        ^
    excellent spread -- each key gets its own stride
```

### The Deletion Problem: Tombstones

In open addressing, you cannot simply mark a slot as empty when deleting. If you do, lookups for keys that probed past that slot will **stop early and fail** to find existing entries.

```
  Insert A at 3, B at 4 (collision, probed from 3):
  ┌───┬───┬───┬───┬───┬───┐
  │   │   │   │ A │ B │   │     B is at 4 because 3 was full
  └───┴───┴───┴───┴───┴───┘

  Delete A, mark slot 3 as EMPTY:
  ┌───┬───┬───┬───┬───┬───┐
  │   │   │   │   │ B │   │     slot 3 is now empty
  └───┴───┴───┴───┴───┴───┘

  Lookup B: hash(B) = 3, slot 3 is EMPTY -> "B not found"  WRONG!
  B is right there at index 4, but we stopped searching at 3.
```

**Solution: Tombstone markers.** Instead of marking deleted slots as empty, mark them as DELETED (tombstone). During lookup, tombstones are treated as "occupied, keep probing." During insert, tombstones can be reused.

```
  Delete A, mark slot 3 as TOMBSTONE (T):
  ┌───┬───┬───┬───┬───┬───┐
  │   │   │   │ T │ B │   │     T = tombstone ("was occupied")
  └───┴───┴───┴───┴───┴───┘

  Lookup B: hash(B) = 3, slot 3 is TOMBSTONE -> keep probing
            slot 4 = B -> FOUND!  Correct!

  Insert C: hash(C) = 3, slot 3 is TOMBSTONE -> can reuse!
  ┌───┬───┬───┬───┬───┬───┐
  │   │   │   │ C │ B │   │     tombstone reclaimed
  └───┴───┴───┴───┴───┴───┘
```

**Downside:** Too many tombstones degrade performance -- the table appears full even when many slots are logically empty. Periodic rehashing clears tombstones.

---

## 🔹 Load Factor

The load factor `alpha` is the ratio of stored elements (`n`) to table size (`m`):

```
  alpha = n / m

  Example: 7 elements in a table of 10 slots -> alpha = 0.7
```

### Impact on Performance

As `alpha` increases, collisions become more frequent and probe sequences grow longer.

```
  Load Factor vs Expected Probes (Open Addressing, Linear Probing)

  alpha │ Avg probes (successful) │ Avg probes (unsuccessful)
  ──────┼─────────────────────────┼───────────────────────────
  0.25  │         1.17            │          1.39
  0.50  │         1.50            │          2.50
  0.70  │         2.17            │          6.06
  0.80  │         3.00            │         13.00
  0.90  │         5.50            │         50.50
  0.95  │        10.50            │        200.50
  ──────┴─────────────────────────┴───────────────────────────

  Visual: probe counts shooting up as table fills

  Probes
    |
 50 |                                                  *
    |                                              *
    |
 10 |                                        *
    |
  5 |                                  *
    |                           *
  2 |                    *
  1 |         *    *
    |____*_______________________________________
     0.1  0.3  0.5  0.6  0.7  0.8  0.9  0.95   alpha
```

### Practical Thresholds

| Strategy        | Typical Rehash Threshold | Notes                                  |
|-----------------|--------------------------|----------------------------------------|
| Open Addressing | alpha > 0.7              | Performance degrades sharply above 0.75 |
| Chaining        | alpha > 1.0 (or higher)  | Can tolerate alpha > 1 because chains grow gracefully |

For open addressing, keeping alpha below 0.7 means most lookups need only 1-2 probes. Above 0.9, the table is nearly unusable.

For chaining, alpha = 1.0 means the average chain length is 1 -- still very fast. Even alpha = 2.0 (average chain length 2) is tolerable in practice.

---

## 🔹 Rehashing

When the load factor exceeds the threshold, the table must **grow and rebuild**.

### Process

1. Allocate a new, larger array (typically 2x the old size, rounded to a prime)
2. Re-hash every existing entry using the new table size
3. Insert each entry into its new position
4. Discard the old array

```
  BEFORE rehash (m=4, n=3, alpha=0.75 -- threshold exceeded!)

  ┌───────┬─────────┬───────┬─────────┐
  │ alice │ (empty) │  bob  │ charlie │
  └───────┴─────────┴───────┴─────────┘
     0        1        2        3

  AFTER rehash (m=8, n=3, alpha=0.375 -- breathing room!)

  Re-hash every key with new modulus:
    hash("alice")   % 8 = 5
    hash("bob")     % 8 = 1
    hash("charlie") % 8 = 7

  ┌───┬─────┬───┬───┬───┬───────┬───┬─────────┐
  │   │ bob │   │   │   │ alice │   │ charlie │
  └───┴─────┴───┴───┴───┴───────┴───┴─────────┘
    0    1    2   3   4     5     6      7
```

### Cost Analysis

```
  Single rehash:  O(n)  -- must touch every element
  Amortized cost: O(1)  -- rehash happens after ~n inserts

  Insert timeline:
  ──────────────────────────────────────────────>
  [1] [1] [1] [1] [n] [1] [1] [1] [1] [1] [1] [1] [1] [2n] ...
                    ^                                     ^
                 rehash                                rehash
                 (cost n)                              (cost 2n)

  Total cost for n inserts: n * O(1) + O(n) rehash = O(n)
  Amortized per insert: O(1)
```

Rehashing also cleans up tombstones in open addressing tables.

---

## 🔹 Impact of Poor Hash Functions

A bad hash function can turn a hash table into an O(n) linked list.

### Example: A Terrible Hash Function

```cpp
// BAD: always returns a constant
int terrible_hash(string key) {
    return 0;  // Every key goes to bucket 0!
}

// Result with chaining: one giant linked list
// ┌───┐
// │ 0 │──> A ──> B ──> C ──> D ──> E ──> F ──> ... ──> NULL
// ├───┤
// │ 1 │──> NULL
// ├───┤
// │ 2 │──> NULL
// └───┘
// Every lookup is O(n). Congratulations, it's a linked list.
```

### Example: A Slightly Less Terrible Hash Function

```cpp
// BAD: only uses first character
int bad_hash(string key, int m) {
    return key[0] % m;
}

// With English words:
// 's' -> bucket 3:  "sun", "sky", "sea", "star", "snow", "sand", ...
// 't' -> bucket 4:  "the", "tree", "top", "ten", "toy", "two", ...
// 'a' -> bucket 7:  "ant", "apple", "air", "arc", ...
//
// Letters are NOT uniformly distributed in English!
// Some buckets overflow while others sit empty.
```

### What Makes a Good Hash Function

A good hash function should:
- **Distribute uniformly** across all buckets
- **Use all bits** of the input (not just the first character)
- **Avalanche** -- a small change in input produces a large change in output
- Be **deterministic** -- same input always gives same output
- Be **fast** to compute

---

## 🔹 Chaining vs Open Addressing: Comparison

| Criteria                | Chaining                         | Open Addressing                  |
|-------------------------|----------------------------------|----------------------------------|
| **Data structure**      | Array of linked lists            | Single flat array                |
| **Memory overhead**     | Extra pointers per entry         | No extra pointers                |
| **Cache performance**   | Poor (pointer chasing)           | Good (sequential memory access)  |
| **Max load factor**     | Can exceed 1.0                   | Must stay below 1.0 (ideally <0.7) |
| **Deletion**            | Simple (unlink node)             | Complex (needs tombstones)       |
| **Clustering**          | No clustering                    | Prone to clustering              |
| **Worst-case lookup**   | O(n) (all in one chain)          | O(n) (all slots probed)          |
| **Implementation**      | Straightforward                  | More complex (probe logic)       |
| **Memory allocation**   | Dynamic (malloc per node)        | Pre-allocated array              |
| **When to use**         | Unknown/variable load, frequent deletes | Known max size, read-heavy workloads |

**In practice:**
- **Chaining** is the most common in standard libraries (Java `HashMap`, Python `dict` pre-3.6) because it's simpler and handles high load gracefully.
- **Open addressing** (specifically Robin Hood or Swiss Table variants) is used when cache performance is critical (C++ `absl::flat_hash_map`, Rust `HashMap`, Python `dict` 3.6+).

---

## 🔹 Common Pitfalls

1. **Forgetting to rehash.** Letting the load factor grow unchecked degrades hash table performance to O(n). Always set a threshold and trigger rehashing.

2. **Deleting in open addressing without tombstones.** This silently breaks lookups for other keys that probed past the deleted slot. Always use tombstone markers.

3. **Using mutable objects as keys.** If a key changes after insertion, its hash changes, and the entry becomes unreachable. Keys must be immutable or treated as immutable once inserted.

4. **Hash function that ignores part of the key.** Using only the first character, or only the length, produces terrible distribution. Use all significant bits of the key.

5. **Table size that is a power of 2 with a bad hash.** If the table size is `2^k`, then `hash % m` only uses the lowest `k` bits. A hash function with poor low-bit entropy causes extreme clustering. Mitigation: use a prime table size, or apply a bit-mixing finalizer.

6. **Accumulating tombstones.** In open-addressing tables with many inserts and deletes, tombstones pile up and lengthen probe sequences even when the actual load is low. Rehash periodically to reclaim them.

7. **Assuming O(1) is guaranteed.** Hash table O(1) is the **average** case with a good hash function. Worst case is always O(n). Adversarial inputs can deliberately trigger worst-case behavior (hash-flooding attacks).

---

## 🔹 Related Concepts

- [[Hash Map and Hash Set]] -- the data structures that rely on collision handling
- [[Singly Linked List]] -- the underlying structure used in chaining
- [[Arrays]] -- the underlying structure used in open addressing
- [[Hash Functions]] -- the quality of the hash function directly determines collision frequency
- [[Amortized Analysis]] -- how rehashing cost is O(1) amortized per insert
- [[Time and Space Complexity]] -- understanding O(1) average vs O(n) worst case
