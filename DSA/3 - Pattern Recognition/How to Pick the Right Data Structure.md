---
tags:
  - algorithms
  - pattern-recognition
  - data-structure
---

## 🔹 Decision Table

| What You Need | Best Data Structure | Time Complexity | Why |
|---|---|---|---|
| Fast lookup by key | **Hash Map** / Hash Set | O(1) average | Direct address via hash function |
| Frequency counting | **Hash Map** | O(1) per update | Key = element, value = count |
| Ordered data + fast search | **Sorted Array + Binary Search** | O(log n) search | Sorted property enables halving |
| Ordered data + fast insert/delete | **Balanced BST** (e.g., `std::set`) | O(log n) all ops | Self-balancing maintains order |
| LIFO processing | **Stack** | O(1) push/pop | Last in, first out |
| FIFO processing | **Queue** | O(1) enqueue/dequeue | First in, first out |
| Fast min/max extraction | **Heap** (Priority Queue) | O(log n) push/pop, O(1) peek | Partial ordering via heap property |
| Top K / Kth largest | **Min-Heap** of size K | O(n log K) total | Keep only K largest elements |
| Median maintenance | **Two Heaps** (max-heap + min-heap) | O(log n) per insert | Split data at median boundary |
| Prefix matching / autocomplete | **Trie** | O(m) per word (m = length) | Character-by-character navigation |
| Range queries (sum, min, max) | **Segment Tree** / BIT | O(log n) query + update | Divide range into segments |
| Disjoint set membership | **Union-Find** (DSU) | ~O(1) amortized | Path compression + union by rank |
| Relationships / connections | **Graph** (adjacency list) | O(V + E) traversal | Flexible relationship modeling |
| Weighted shortest path | **Graph + Min-Heap** | O(E log V) Dijkstra | Greedy extraction of nearest node |
| Sequential + random access | **Array** | O(1) access, O(n) insert | Contiguous memory, cache-friendly |
| Frequent insert/delete at ends | **Deque** / Linked List | O(1) at ends | No shifting required |
| Sliding window min/max | **Monotonic Deque** | O(1) amortized per op | Maintains order invariant |

---

## 🔹 When Two Data Structures Compete

| Scenario | Array | Linked List | Winner & Why |
|---|---|---|---|
| Random access by index | O(1) | O(n) | **Array** — direct indexing |
| Insert at front | O(n) | O(1) | **Linked List** — no shifting |
| Cache performance | Excellent | Poor | **Array** — contiguous memory |
| Unknown size, many inserts | Amortized O(1) | O(1) | **Either** — arrays resize well in practice |

| Scenario | Hash Map | BST (e.g., `std::map`) | Winner & Why |
|---|---|---|---|
| Pure lookup/insert | O(1) avg | O(log n) | **Hash Map** — faster average case |
| Need sorted order | N/A (unsorted) | O(log n) iteration | **BST** — maintains order |
| Range queries (all keys in [a,b]) | O(n) | O(log n + k) | **BST** — `lower_bound`/`upper_bound` |
| Worst case guarantee | O(n) worst | O(log n) guaranteed | **BST** — no hash collision risk |

---

## 🔹 The "What Do I Need?" Quick Check

Ask yourself these questions in order:

1. **Do I need key-value lookup?**
   - Yes, unordered → `unordered_map` / `HashMap`
   - Yes, ordered → `map` / `TreeMap`
   - Just existence → `unordered_set` / `HashSet`

2. **Do I need to repeatedly extract min or max?**
   - Yes → `priority_queue` / Heap
   - Need both min AND max → Two heaps or balanced BST

3. **Do I need LIFO or FIFO?**
   - LIFO → `stack`
   - FIFO → `queue`

4. **Do I need fast range operations?**
   - Sum/min/max over range → Segment Tree or BIT
   - Just prefix sums → Prefix Sum Array

5. **Do I need to group/merge elements?**
   - Connected components → Union-Find
   - Group by property → Hash Map with lists

---

## 🔹 Complexity Cheat Sheet by Data Structure

| Data Structure | Access | Search | Insert | Delete | Space |
|---|---|---|---|---|---|
| Array | O(1) | O(n) | O(n) | O(n) | O(n) |
| Sorted Array | O(1) | O(log n) | O(n) | O(n) | O(n) |
| Linked List | O(n) | O(n) | O(1)* | O(1)* | O(n) |
| Hash Map/Set | — | O(1) avg | O(1) avg | O(1) avg | O(n) |
| BST (balanced) | — | O(log n) | O(log n) | O(log n) | O(n) |
| Heap | peek: O(1) | O(n) | O(log n) | O(log n) | O(n) |
| Stack | top: O(1) | O(n) | O(1) | O(1) | O(n) |
| Queue | front: O(1) | O(n) | O(1) | O(1) | O(n) |
| Trie | — | O(m) | O(m) | O(m) | O(ALPHABET * m * n) |
| Union-Find | — | ~O(1) | ~O(1) | — | O(n) |

*\* O(1) insert/delete at known position; O(n) to find the position first.*

---

## 🔹 Related

- [[Pattern - Hash Map]] — deep dive on hash map usage patterns
- [[Pattern - Stack]] — when and how to use stacks
- [[How to Pick the Right Algorithm]] — pair the right structure with the right technique
- [[Constraint Analysis]] — data structure choice affects overall complexity
