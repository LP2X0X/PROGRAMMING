---
tags:
  - algorithms
  - pattern-recognition
  - flowchart
---

## 🔹 The Master Decision Tree

When you see a new problem, walk through these questions **in order**. Each branch narrows the technique space.

```
START: Read the problem
  │
  ├─ Is input a TREE or GRAPH structure?
  │   ├─ Tree? ──────────────────────────→ DFS/BFS Recursion (see [[Pattern - Tree]])
  │   └─ Graph? ─────────────────────────→ BFS / DFS / Topological Sort
  │       ├─ Shortest path (unweighted)? → BFS
  │       ├─ Shortest path (weighted)?   → Dijkstra / Bellman-Ford
  │       ├─ Detect cycle?               → DFS with visited states
  │       ├─ Dependency ordering?        → Topological Sort
  │       └─ Connected components?       → Union-Find / DFS
  │                                        (see [[Pattern - Graph]])
  │
  ├─ Is the input SORTED (or can sorting help)?
  │   ├─ Searching for a target?         → Binary Search (see [[Pattern - Binary Search]])
  │   ├─ Finding pairs/triplets?         → Two Pointers (see [[Pattern - Two Pointers]])
  │   └─ Merging sorted data?            → Two Pointers / Merge technique
  │
  ├─ Is this about a SUBARRAY or SUBSTRING?
  │   ├─ Contiguous + fixed size?        → Fixed Sliding Window
  │   ├─ Contiguous + condition-based?   → Variable Sliding Window
  │   │                                    (see [[Pattern - Sliding Window]])
  │   ├─ Max sum subarray?               → Kadane's Algorithm
  │   └─ Prefix-related?                 → Prefix Sum (see [[Pattern - Array and String]])
  │
  ├─ Need FAST LOOKUP / COUNTING?
  │   ├─ "Does X exist?"                 → Hash Set
  │   ├─ "How many times?"               → Hash Map
  │   └─ "Find pair that sums to K"      → Hash Map (see [[Pattern - Hash Map]])
  │
  ├─ MATCHING / NESTING / ORDERING?
  │   ├─ Brackets / parentheses?         → Stack
  │   ├─ Next greater/smaller element?   → Monotonic Stack
  │   └─ Expression evaluation?          → Stack (see [[Pattern - Stack]])
  │
  ├─ OPTIMIZATION problem ("min/max/best")?
  │   ├─ Overlapping subproblems?        → DP (see [[Pattern - Dynamic Programming]])
  │   ├─ Local choice = global optimal?  → Greedy (see [[Pattern - Greedy]])
  │   └─ "Minimum X satisfying Y"?       → Binary Search on Answer
  │
  ├─ GENERATE ALL possibilities?
  │   ├─ Permutations / Combinations?    → Backtracking
  │   ├─ Subsets?                         → Backtracking or Bitmask
  │   └─ Constraint satisfaction?        → Backtracking (see [[Pattern - Backtracking]])
  │
  ├─ Need REPEATED MIN/MAX extraction?
  │   └─ Top K / Kth largest / merge K?  → Heap (Priority Queue)
  │
  └─ SPECIAL PATTERNS
      ├─ Grid problem?                   → Treat as implicit graph → BFS/DFS
      ├─ Interval merging / scheduling?  → Sort + Greedy / Sweep Line
      ├─ Bit manipulation clue?          → XOR tricks, bitmask
      └─ String prefix matching?         → Trie
```

---

## 🔹 Comprehensive Decision Table

Quick-scan reference. Match your problem's **keywords** and **structure** to the right technique.

| Keyword / Signal | Technique | Link |
|---|---|---|
| "sorted array", "find target" | Binary Search | [[Pattern - Binary Search]] |
| "pair with sum", "two sum" | Hash Map or Two Pointers | [[Pattern - Hash Map]], [[Pattern - Two Pointers]] |
| "three sum", "triplet" | Sort + Two Pointers | [[Pattern - Two Pointers]] |
| "subarray of size K" | Fixed Sliding Window | [[Pattern - Sliding Window]] |
| "longest substring with at most..." | Variable Sliding Window | [[Pattern - Sliding Window]] |
| "maximum subarray sum" | Kadane's Algorithm | [[Pattern - Array and String]] |
| "contiguous subarray sum = K" | Prefix Sum + Hash Map | [[Pattern - Array and String]] |
| "valid parentheses", "matching brackets" | Stack | [[Pattern - Stack]] |
| "next greater element" | Monotonic Stack | [[Pattern - Stack]] |
| "number of islands", "flood fill" | BFS / DFS on Grid | [[Pattern - Graph]] |
| "shortest path" | BFS (unweighted) / Dijkstra (weighted) | [[Pattern - Graph]] |
| "course schedule", "task ordering" | Topological Sort | [[Pattern - Graph]] |
| "lowest common ancestor" | Tree DFS | [[Pattern - Tree]] |
| "serialize / deserialize tree" | Tree Traversal | [[Pattern - Tree]] |
| "minimum cost to...", "number of ways" | Dynamic Programming | [[Pattern - Dynamic Programming]] |
| "longest increasing subsequence" | DP (or DP + Binary Search) | [[Pattern - Dynamic Programming]] |
| "coin change", "knapsack" | Dynamic Programming | [[Pattern - Dynamic Programming]] |
| "minimum intervals to cover" | Greedy (sort + sweep) | [[Pattern - Greedy]] |
| "activity selection", "meeting rooms" | Greedy / Sort + Sweep | [[Pattern - Greedy]] |
| "all permutations", "all subsets" | Backtracking | [[Pattern - Backtracking]] |
| "Sudoku", "N-Queens" | Backtracking | [[Pattern - Backtracking]] |
| "top K elements", "K-th largest" | Heap | [[How to Pick the Right Data Structure]] |
| "merge K sorted lists" | Heap + Merge | [[How to Pick the Right Data Structure]] |
| "frequency count", "anagram" | Hash Map | [[Pattern - Hash Map]] |
| "remove duplicates" | Hash Set or Two Pointers | [[Pattern - Hash Map]], [[Pattern - Two Pointers]] |
| "palindrome" | Two Pointers (outside-in) | [[Pattern - Two Pointers]] |
| "in-place modification" | Two Pointers (read/write) | [[Pattern - Two Pointers]] |
| "prefix matching", "autocomplete" | Trie | [[How to Pick the Right Data Structure]] |
| "detect cycle in linked list" | Fast/Slow Pointers | [[Pattern - Two Pointers]] |
| "median of data stream" | Two Heaps | [[How to Pick the Right Data Structure]] |

---

## 🔹 The 3-Step Recognition Process

1. **Read the constraints** — determine required time complexity (see [[Constraint Analysis]])
2. **Identify the input structure** — array, string, tree, graph, grid, intervals?
3. **Match keywords and patterns** — use the decision tree and table above

> [!tip] When You're Stuck
> If no single pattern fits, the problem likely combines two patterns. Common combos:
> - **Sort + Two Pointers** (3Sum)
> - **Hash Map + Prefix Sum** (subarray sum equals K)
> - **Binary Search + Greedy/DP** (minimize the maximum)
> - **BFS + Hash Map** (word ladder)
> - **Stack + Hash Map** (next greater element with frequency)

---

## 🔹 Related

- [[Constraint Analysis]] — use input size to narrow technique choices
- [[How to Pick the Right Data Structure]] — when the bottleneck is the data structure
- [[How to Pick the Right Algorithm]] — systematic technique selection
