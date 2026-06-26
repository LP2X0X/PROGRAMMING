---
tags:
  - algorithms
  - big-o
  - reference
---

# Big O Cheat Sheet

> Quick-reference page for time and space complexities. Open this when you need to look up a complexity fast. For deep explanations, see [[Big O - Definition]].

---

## 🔹 Data Structure Operations

| Data Structure | Access | Search | Insert | Delete | Notes |
|---|---|---|---|---|---|
| **Array** | O(1) | O(n) | O(n) | O(n) | Insert/delete shift elements |
| **Sorted Array** | O(1) | O(log n) | O(n) | O(n) | Binary search for lookup |
| **Dynamic Array** (vector) | O(1) | O(n) | O(1)* | O(n) | *Amortized for push_back |
| **Singly Linked List** | O(n) | O(n) | O(1)** | O(1)** | **At head; O(n) to find position |
| **Doubly Linked List** | O(n) | O(n) | O(1)** | O(1)** | **Given the node; O(n) to find |
| **Stack** (array/LL) | O(n) | O(n) | O(1) | O(1) | Push/pop at top only |
| **Queue** (LL/circular) | O(n) | O(n) | O(1) | O(1) | Enqueue back, dequeue front |
| **Hash Table** | - | O(1)* | O(1)* | O(1)* | *Average; O(n) worst (collisions) |
| **BST** (balanced) | O(log n) | O(log n) | O(log n) | O(log n) | Unbalanced degrades to O(n) |
| **BST** (unbalanced worst) | O(n) | O(n) | O(n) | O(n) | When tree becomes a chain |
| **Min/Max Heap** | O(1)*** | O(n) | O(log n) | O(log n) | ***O(1) for min or max only |
| **Trie** | - | O(L) | O(L) | O(L) | L = length of key/word |

```
Quick visual:

                    Access    Search    Insert    Delete
Array               O(1)      O(n)      O(n)      O(n)
Hash Table           -        O(1)*     O(1)*     O(1)*    <-- fastest for search
Balanced BST        O(log n)  O(log n)  O(log n)  O(log n) <-- all-rounder
Heap                O(1)***   O(n)      O(log n)  O(log n) <-- best for min/max
Linked List         O(n)      O(n)      O(1)**    O(1)**   <-- best for front insert
```

---

## 🔹 Sorting Algorithms

| Algorithm | Best | Average | Worst | Space | Stable? | In-Place? |
|---|---|---|---|---|---|---|
| **Bubble Sort** | O(n) | O(n^2) | O(n^2) | O(1) | Yes | Yes |
| **Selection Sort** | O(n^2) | O(n^2) | O(n^2) | O(1) | No | Yes |
| **Insertion Sort** | O(n) | O(n^2) | O(n^2) | O(1) | Yes | Yes |
| [[Merge Sort]] | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | No |
| [[Quick Sort]] | O(n log n) | O(n log n) | O(n^2) | O(log n) | No | Yes |
| **Heap Sort** | O(n log n) | O(n log n) | O(n log n) | O(1) | No | Yes |
| **Counting Sort** | O(n + k) | O(n + k) | O(n + k) | O(k) | Yes | No |
| **Radix Sort** | O(d(n+k)) | O(d(n+k)) | O(d(n+k)) | O(n+k) | Yes | No |

Where k = range of values, d = number of digits.

> [!tip] Quick Pick
> - Need guaranteed O(n log n)? --> [[Merge Sort]] or Heap Sort
> - General purpose on arrays? --> [[Quick Sort]] (randomized pivot)
> - Small array or nearly sorted? --> Insertion Sort
> - Need stability? --> [[Merge Sort]]
> - See [[Sorting Comparison]] for the full decision flowchart

---

## 🔹 Common Algorithm Complexities

| Algorithm | Time | Space | Notes |
|---|---|---|---|
| **[[Binary Search]]** | O(log n) | O(1) iter / O(log n) rec | Array must be sorted |
| **Linear Search** | O(n) | O(1) | Unsorted data |
| **BFS** (graph) | O(V + E) | O(V) | Queue + visited set |
| **DFS** (graph) | O(V + E) | O(V) | Stack/recursion + visited |
| **Dijkstra** (min-heap) | O((V+E) log V) | O(V) | Non-negative weights only |
| **Bellman-Ford** | O(V * E) | O(V) | Handles negative weights |
| **Floyd-Warshall** | O(V^3) | O(V^2) | All-pairs shortest path |
| **Topological Sort** | O(V + E) | O(V) | DAGs only (Kahn's or DFS) |
| **Kruskal's MST** | O(E log E) | O(V) | Union-Find needed |
| **Prim's MST** | O((V+E) log V) | O(V) | Min-heap variant |
| **KMP String Match** | O(n + m) | O(m) | n = text, m = pattern |
| **Two Pointers** | O(n) | O(1) | Sorted array problems |
| **Sliding Window** | O(n) | O(1)~O(k) | Subarray/substring problems |

---

## 🔹 Dynamic Programming Patterns

| Pattern | Time | Space | Can Optimize Space? |
|---|---|---|---|
| **1D DP** (fibonacci, climbing stairs) | O(n) | O(n) | Yes --> O(1) with rolling vars |
| **2D DP** (LCS, edit distance, knapsack) | O(n * m) | O(n * m) | Often --> O(min(n,m)) with row reuse |
| **Interval DP** (matrix chain mult) | O(n^3) | O(n^2) | Rarely |
| **Bitmask DP** (TSP, assignments) | O(2^n * n) | O(2^n) | No |
| **Tree DP** | O(n) | O(n) | No |
| **Knapsack** (0/1) | O(n * W) | O(n * W) | Yes --> O(W) with 1D array |

See [[Memoization vs Tabulation]] for top-down vs bottom-up approaches.

---

## 🔹 Constraint --> Required Complexity

Use the input size `n` from a problem to determine which Big O you need. Assuming ~10^8 operations per second as the safe limit. See [[Constraint Analysis]] for full details.

| Constraint (max n) | Max Complexity | Typical Approach |
|---|---|---|
| n <= 10 | O(n!) | Brute force, try all permutations |
| n <= 20 | O(2^n) | Bitmask DP, backtracking with pruning |
| n <= 100 | O(n^4) | Rare; brute force with 4 loops |
| n <= 500 | O(n^3) | Floyd-Warshall, triple loop |
| n <= 5,000 | O(n^2) | Nested loops, simple DP |
| n <= 100,000 | O(n log n) | Sort-based, divide & conquer, segment tree |
| n <= 1,000,000 | O(n) | Hash map, prefix sum, two pointers |
| n <= 10^9 | O(log n) or O(sqrt(n)) | Binary search, math |
| n <= 10^18 | O(log n) | Binary search, fast exponentiation, math |

> [!warning] These Are Guidelines
> Real competitive programming time limits vary. When in doubt, estimate: if `n = 10^5` and your algorithm is O(n^2), that's 10^10 operations -- way too slow.

---

## 🔹 One-Liner Rules

**Drop constants:**
`O(2n)` = `O(n)` | `O(100)` = `O(1)` | `O(n/2)` = `O(n)`

**Drop lower-order terms:**
`O(n^2 + n)` = `O(n^2)` | `O(n + log n)` = `O(n)` | `O(2^n + n^3)` = `O(2^n)`

**Nested = Multiply:**
Loop inside loop: `O(n) * O(m)` = `O(n*m)`

**Sequential = Add (then take max):**
Loop after loop: `O(n) + O(m)` = `O(n + m)`, and if `n == m`, simplify to `O(n)`

**Different inputs = Different variables:**
`O(a + b)` stays as is -- don't assume `a == b`

**Logarithm base doesn't matter:**
`O(log_2 n)` = `O(log_10 n)` = `O(ln n)` = `O(log n)` (differ by constant factor)

---

## 🔹 Hidden Complexity Traps

| Operation | What It Looks Like | Actual Cost | Why |
|---|---|---|---|
| `string += char` in loop | O(n) total | O(n^2) total | Each += copies the whole string |
| `vector.insert(begin, x)` | O(1) | O(n) | Shifts all elements right |
| `list.contains(x)` (unsorted) | O(1) | O(n) | Linear scan |
| `set.contains(x)` (tree-based) | O(1) | O(log n) | Tree traversal |
| `sort()` inside a loop | O(n log n) | O(n^2 log n) | Sorting n times |
| Recursive call count | "just a few" | O(2^n) | Branching recursion |

---

## 🔹 Space Complexity Quick Reference

| Pattern | Space |
|---|---|
| Fixed number of variables | O(1) |
| Single array of size n | O(n) |
| Hash map of n elements | O(n) |
| 2D matrix n x m | O(n * m) |
| Recursion depth d | O(d) stack frames |
| [[Merge Sort]] temp array | O(n) |
| [[Quick Sort]] recursion | O(log n) avg, O(n) worst |
| BFS queue on graph | O(V) |
| DFS stack on tree | O(h) where h = height |
| Bitmask DP (2^n states) | O(2^n) |

---

## 🔹 Master Theorem Quick Reference

For recurrences of the form: `T(n) = a * T(n/b) + O(n^d)`

| Compare | Result |
|---|---|
| `d < log_b(a)` | T(n) = O(n^(log_b(a))) |
| `d == log_b(a)` | T(n) = O(n^d * log n) |
| `d > log_b(a)` | T(n) = O(n^d) |

**Common results:**

| Recurrence | a | b | d | Case | Result |
|---|---|---|---|---|---|
| T(n) = 2T(n/2) + O(n) | 2 | 2 | 1 | 2 | **O(n log n)** -- [[Merge Sort]] |
| T(n) = T(n/2) + O(1) | 1 | 2 | 0 | 2 | **O(log n)** -- [[Binary Search]] |
| T(n) = 2T(n/2) + O(1) | 2 | 2 | 0 | 1 | **O(n)** -- tree traversal |
| T(n) = 2T(n/2) + O(n^2) | 2 | 2 | 2 | 3 | **O(n^2)** |
| T(n) = 4T(n/2) + O(n) | 4 | 2 | 1 | 1 | **O(n^2)** |
| T(n) = 3T(n/2) + O(n) | 3 | 2 | 1 | 1 | **O(n^1.585)** |

---

**See also:** [[Big O - Definition]] | [[Constraint Analysis]] | [[Sorting Comparison]]
