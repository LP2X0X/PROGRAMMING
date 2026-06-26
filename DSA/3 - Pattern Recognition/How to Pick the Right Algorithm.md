---
tags:
  - algorithms
  - pattern-recognition
  - technique
---

## 🔹 Master Decision Table

| Problem Type | Technique | Key Signal | Complexity | Link |
|---|---|---|---|---|
| Search in sorted data | **Binary Search** | sorted + "find" / "search" | O(log n) | [[Pattern - Binary Search]] |
| Find min/max satisfying condition | **Binary Search on Answer** | monotonic answer space + feasibility check | O(log(range) * check) | [[Pattern - Binary Search]] |
| Subarray/substring with constraint | **Sliding Window** | contiguous + "longest" / "shortest" / "at most K" | O(n) | [[Pattern - Sliding Window]] |
| Pair/triplet in sorted data | **Two Pointers** | sorted + pair + sum/target | O(n) or O(n^2) | [[Pattern - Two Pointers]] |
| In-place array manipulation | **Two Pointers** (read/write) | remove/partition without extra space | O(n) | [[Pattern - Two Pointers]] |
| Cycle detection in linked list | **Fast/Slow Pointers** | linked list + cycle | O(n) | [[Pattern - Two Pointers]] |
| Lookup / counting / grouping | **Hash Map** | frequency, existence, pairing | O(n) | [[Pattern - Hash Map]] |
| Nesting / matching / LIFO | **Stack** | brackets, recursion simulation | O(n) | [[Pattern - Stack]] |
| Next greater/smaller element | **Monotonic Stack** | "next greater", "previous smaller" | O(n) | [[Pattern - Stack]] |
| Shortest path (unweighted) | **BFS** | graph + shortest + equal weights | O(V + E) | [[Pattern - Graph]] |
| Shortest path (weighted, non-negative) | **Dijkstra** | graph + shortest + positive weights | O(E log V) | [[Pattern - Graph]] |
| Shortest path (negative weights) | **Bellman-Ford** | graph + negative edges | O(V * E) | [[Pattern - Graph]] |
| All-pairs shortest path | **Floyd-Warshall** | small graph + all pairs | O(V^3) | [[Pattern - Graph]] |
| Explore all paths / connectivity | **DFS** | graph + "all paths" / "connected" | O(V + E) | [[Pattern - Graph]] |
| Dependency ordering | **Topological Sort** | DAG + prerequisites / ordering | O(V + E) | [[Pattern - Graph]] |
| Connected components | **Union-Find** or DFS | group elements, "number of islands" | O(V + E) | [[Pattern - Graph]] |
| Tree traversal | **DFS** (pre/in/post) or **BFS** | tree + path / depth / level | O(n) | [[Pattern - Tree]] |
| Lowest common ancestor | **DFS + Parent tracking** | tree + "common ancestor" | O(n) | [[Pattern - Tree]] |
| Optimization with overlapping subs | **Dynamic Programming** | "minimum cost", "number of ways", optimal + subproblems | varies | [[Pattern - Dynamic Programming]] |
| Optimization with greedy choice | **Greedy** | "minimum X to cover", intervals, local = global | O(n log n) | [[Pattern - Greedy]] |
| Generate all combos/perms | **Backtracking** | "all valid", "generate all", subsets | O(2^n) or O(n!) | [[Pattern - Backtracking]] |
| Constraint satisfaction | **Backtracking** with pruning | Sudoku, N-Queens, word search | O(k^n) pruned | [[Pattern - Backtracking]] |

---

## 🔹 When Two Techniques Compete

Some problems can be solved by multiple techniques. Here's how to pick:

| Scenario | Option A | Option B | How to Choose |
|---|---|---|---|
| "Two Sum" | Hash Map O(n) | Sort + Two Pointers O(n log n) | Hash Map is faster; Two Pointers if already sorted |
| Subarray sum = K | Sliding Window | Prefix Sum + Hash Map | Sliding Window only works for positive values; Prefix Sum handles negatives |
| Optimization problem | DP | Greedy | Try Greedy first (simpler); if greedy fails, DP. Prove greedy with exchange argument |
| Graph shortest path | BFS | Dijkstra | BFS if all edges weight 1; Dijkstra if variable positive weights |
| "Longest increasing subsequence" | O(n^2) DP | O(n log n) DP + Binary Search | Depends on n (see [[Constraint Analysis]]) |
| "Find Kth largest" | Sort O(n log n) | Quickselect O(n) avg | Quickselect for single query; Sort if multiple queries |

---

## 🔹 Combination Patterns

Many medium/hard problems combine two techniques. Recognize the combo:

| Combination | Example Problem | Why Both? |
|---|---|---|
| **Sort + Two Pointers** | 3Sum, Container With Most Water | Sort enables pointer logic |
| **Hash Map + Prefix Sum** | Subarray Sum Equals K | Hash map stores prefix sums for O(1) lookups |
| **Binary Search + Greedy/DP** | Minimize Maximum Page Allocation | BS picks candidate answer, greedy validates |
| **BFS + Hash Map** | Word Ladder | BFS explores paths, hash map checks valid words |
| **DFS + Memoization** | Grid pathfinding with obstacles | DFS explores, memo avoids recomputation (= top-down DP) |
| **Stack + Hash Map** | Next greater element across arrays | Stack finds next greater, hash map maps results |
| **Heap + Two Pointers** | Merge K sorted lists | Heap picks minimum, pointers track position per list |
| **Graph + DP** | Shortest path with state (Dijkstra + DP) | State-space graph with memoized distances |

---

## 🔹 The Elimination Process

When stuck, eliminate techniques that **can't** work:

1. **Check constraints** → eliminates techniques too slow (see [[Constraint Analysis]])
2. **Check input structure** → not a graph? eliminate graph algorithms
3. **Check "all" vs "optimal"** → "find all" = backtracking; "find best" = DP or greedy
4. **Check subproblem overlap** → no overlap = greedy might work; overlap = DP
5. **Check if sorted** → not sorted and can't sort? eliminate two pointers, binary search

---

## 🔹 Related

- [[Problem Solving Flowchart]] — visual decision tree
- [[How to Pick the Right Data Structure]] — pair structure with technique
- [[Constraint Analysis]] — use input size to narrow options
