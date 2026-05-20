---
tags:
 - csharp
 - collections
 - generics
---

## Why Generic Collections?

Generic collections solve the two biggest problems of `System.Collections`:

1. **No boxing/unboxing** — value types are stored as-is, not wrapped in `object`. Much better performance.
2. **Compile-time type safety** — the compiler catches type mismatches before runtime.

```csharp
// Non-generic — compiles fine, blows up at runtime
ArrayList list = new ArrayList();
list.Add(42);
list.Add("oops");
int x = (int)list[1]; // InvalidCastException at runtime

// Generic — compiler catches the mistake immediately
List<int> list = new List<int>();
list.Add(42);
// list.Add("oops"); // Compile error: cannot convert string to int
```

---

## Key Collection Types

### `List<T>` — Dynamic Array

The most commonly used collection. Internally backed by an array that doubles in capacity when full.

```csharp
List<string> names = new List<string>();
names.Add("Alice");
names.Add("Bob");
names.AddRange(new[] { "Charlie", "Diana" });

names.Remove("Bob");
names.Insert(1, "Eve");

string first = names[0];         // indexed access O(1)
bool has = names.Contains("Eve"); // linear search O(n)
names.Sort();                     // in-place sort

// Collection initializer syntax
List<int> nums = new List<int> { 1, 2, 3, 4, 5 };

// Useful methods
int idx = nums.IndexOf(3);              // 2
List<int> evens = nums.FindAll(n => n % 2 == 0); // [2, 4]
bool any = nums.Exists(n => n > 10);    // false
nums.ForEach(n => Console.Write(n + " "));
```

| Operation              | Time Complexity        |
| ---------------------- | ---------------------- |
| Index access `list[i]` | O(1)                   |
| Add (at end)           | O(1) amortized         |
| Insert (at index)      | O(n) — shifts elements |
| Remove (by value)      | O(n) — search + shift  |
| Contains / IndexOf     | O(n)                   |
| Sort                   | O(n log n)             |

---

### `Dictionary<TKey, TValue>` — Hash Map

Stores key-value pairs with fast lookup by key. Keys must be unique.

```csharp
Dictionary<string, int> ages = new Dictionary<string, int>
{
    ["Alice"] = 30,
    ["Bob"] = 25,
    ["Charlie"] = 35
};

ages.Add("Diana", 28);
ages["Alice"] = 31;           // update existing

int bobAge = ages["Bob"];     // 25 — throws if key missing

// Safe lookup
if (ages.TryGetValue("Eve", out int eveAge))
    Console.WriteLine(eveAge);
else
    Console.WriteLine("Not found");

// Iteration
foreach (KeyValuePair<string, int> kvp in ages)
    Console.WriteLine($"{kvp.Key}: {kvp.Value}");

// Check existence
bool hasKey = ages.ContainsKey("Bob");     // true
bool hasVal = ages.ContainsValue(25);      // true (O(n) — scans all values)

ages.Remove("Charlie");
```

| Operation | Time Complexity |
|---|---|
| Lookup by key `dict[key]` | O(1) average |
| Add / Update | O(1) average |
| Remove by key | O(1) average |
| ContainsKey | O(1) average |
| ContainsValue | O(n) — no index on values |

> [!warning]
> Key types must implement `GetHashCode()` and `Equals()` correctly. Built-in types (`string`, `int`, etc.) already do. For custom classes, override both or provide an `IEqualityComparer<T>`.

---

### `Queue<T>` — FIFO (First-In, First-Out)

Think of a line at a store — first person in line gets served first.

```csharp
Queue<string> orders = new Queue<string>();
orders.Enqueue("Order A");  // add to back
orders.Enqueue("Order B");
orders.Enqueue("Order C");

string next = orders.Dequeue(); // "Order A" — removes from front
string peek = orders.Peek();    // "Order B" — look without removing

Console.WriteLine(orders.Count); // 2

// Process all items
while (orders.Count > 0)
{
    Console.WriteLine(orders.Dequeue());
}
```

**Use cases:** task scheduling, BFS (breadth-first search), message processing, print queues.

---

### `Stack<T>` — LIFO (Last-In, First-Out)

Think of a stack of plates — you take the top plate off first.

```csharp
Stack<string> history = new Stack<string>();
history.Push("Page 1");  // push on top
history.Push("Page 2");
history.Push("Page 3");

string current = history.Pop();  // "Page 3" — removes top
string top = history.Peek();     // "Page 2" — look without removing
```

**Use cases:** undo/redo, browser back button, DFS (depth-first search), expression parsing.

---

### `HashSet<T>` — Unique Elements Only

An unordered collection that guarantees no duplicates. Very fast membership testing.

```csharp
HashSet<int> set = new HashSet<int> { 1, 2, 3, 4, 5 };

set.Add(3);    // returns false — already exists
set.Add(6);    // returns true — added
set.Remove(1);

bool exists = set.Contains(3); // O(1)

// Set operations
HashSet<int> other = new HashSet<int> { 4, 5, 6, 7, 8 };

set.UnionWith(other);         // set = {2, 3, 4, 5, 6, 7, 8}
set.IntersectWith(other);     // set = elements in both
set.ExceptWith(other);        // set = elements in set but not in other
set.SymmetricExceptWith(other); // elements in either but not both

bool isSubset = set.IsSubsetOf(other);
bool overlaps = set.Overlaps(other);
```

**Use cases:** removing duplicates, fast lookup when you only care about existence (not key-value), set math.

---

### `LinkedList<T>` — Doubly Linked List

Each element (node) holds a reference to the next and previous node. Good for frequent insertions/removals in the middle.

```csharp
LinkedList<string> list = new LinkedList<string>();

list.AddLast("A");
list.AddLast("C");
LinkedListNode<string> nodeC = list.Find("C");
list.AddBefore(nodeC, "B");  // A -> B -> C

list.AddFirst("Z");          // Z -> A -> B -> C
list.RemoveFirst();           // A -> B -> C

// Traverse
LinkedListNode<string> node = list.First;
while (node != null)
{
    Console.Write(node.Value + " ");
    node = node.Next;
}
```

| Operation | Time Complexity |
|---|---|
| AddFirst / AddLast | O(1) |
| Insert before/after a known node | O(1) |
| Remove a known node | O(1) |
| Find by value | O(n) |
| Index access | Not supported — must traverse |

> [!note]
> In practice, `List<T>` beats `LinkedList<T>` in most scenarios due to CPU cache locality. Use `LinkedList<T>` only when you need constant-time inserts/removals at known positions.

---

### `SortedList<TKey, TValue>` and `SortedDictionary<TKey, TValue>`

Both store key-value pairs sorted by key, but with different tradeoffs:

| Feature | `SortedList<K,V>` | `SortedDictionary<K,V>` |
|---|---|---|
| Internal structure | Two parallel arrays (keys + values) | Red-black tree |
| Memory | Less | More (tree node overhead) |
| Indexed access by position | Yes — `list.Keys[0]` | No |
| Insert/Remove | O(n) — shifts array | O(log n) |
| Lookup by key | O(log n) — binary search | O(log n) — tree walk |
| Best when | Data loaded once, queried often | Frequent inserts/removals |

```csharp
SortedDictionary<string, int> scores = new SortedDictionary<string, int>
{
    ["Charlie"] = 85,
    ["Alice"] = 92,
    ["Bob"] = 78
};

// Iteration is always in key order
foreach (var kvp in scores)
    Console.WriteLine($"{kvp.Key}: {kvp.Value}");
// Alice: 92
// Bob: 78
// Charlie: 85
```

---

## Key Generic Interfaces

```
IEnumerable<T>
   └── ICollection<T>
          ├── IList<T>          (List<T>, arrays)
          ├── IDictionary<K,V>  (Dictionary<K,V>, SortedDictionary<K,V>)
          └── ISet<T>           (HashSet<T>, SortedSet<T>)
```

- **`IEnumerable<T>`** — `foreach` support via `GetEnumerator()`.
- **`ICollection<T>`** — `Count`, `Add`, `Remove`, `Contains`, `Clear`.
- **`IList<T>`** — adds indexed access `this[int]`, `Insert`, `RemoveAt`.
- **`IDictionary<TKey, TValue>`** — adds keyed access `this[TKey]`, `Keys`, `Values`.
- **`ISet<T>`** — adds set operations: `UnionWith`, `IntersectWith`, `ExceptWith`.

> [!tip]
> Prefer declaring variables and parameters with the **interface type** rather than the concrete type when possible. This makes code easier to change later.
> ```csharp
> // Good — callers don't depend on List<T> internals
> IList<string> names = new List<string>();
> 
> // Good — only need iteration? use the minimal interface
> IEnumerable<string> names = GetNames();
> ```

---

## Quick Reference — Choosing the Right Collection

| Need | Use |
|---|---|
| Ordered list with indexed access | `List<T>` |
| Key-value lookup | `Dictionary<TKey, TValue>` |
| Unique elements, fast membership test | `HashSet<T>` |
| FIFO processing | `Queue<T>` |
| LIFO / undo operations | `Stack<T>` |
| Sorted key-value data | `SortedDictionary<TKey, TValue>` |
| Frequent insert/remove in middle | `LinkedList<T>` |
| Thread-safe collections | `System.Collections.Concurrent` namespace |

---

![[Pasted image 20240614100737.png|center|700]]
![[Pasted image 20240614100640.png|center|700]]