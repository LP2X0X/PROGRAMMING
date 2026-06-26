---
tags:
  - algorithms
  - pattern-recognition
  - constraints
---

## 🔹 The Core Idea

Competitive programming problems and coding interviews almost always state the **input size** (`n`). This single number tells you the maximum time complexity your solution can have. Most online judges allow roughly **10^8 operations per second**. Work backwards from the time limit (usually 1-2 seconds) to figure out what algorithmic complexity you need.

---

## 🔹 Constraint-to-Complexity Reference Table

| Input Size (n) | Max Complexity | Typical Techniques |
|---|---|---|
| n <= 10 | O(n!) | Brute force, permutations, [[Pattern - Backtracking]] |
| n <= 20 | O(2^n) | Bitmask DP, backtracking with pruning |
| n <= 25 | O(2^(n/2)) | Meet in the middle |
| n <= 100 | O(n^4) | DP with multiple dimensions |
| n <= 500 | O(n^3) | Triple nested loops, Floyd-Warshall, some DP |
| n <= 5,000 | O(n^2) | Nested loops, 2D DP, insertion sort |
| n <= 100,000 (10^5) | O(n log n) | Sorting, [[Pattern - Binary Search]], merge sort, heap operations, balanced BST, segment tree |
| n <= 1,000,000 (10^6) | O(n) | Linear scan, [[Pattern - Hash Map]], [[Pattern - Two Pointers]], [[Pattern - Sliding Window]], prefix sum, counting sort |
| n <= 10^9 | O(log n) or O(sqrt(n)) | [[Pattern - Binary Search]], math, binary search on answer |
| n <= 10^18 | O(log n) | Binary search, math, fast exponentiation |

---

## 🔹 How to Use This Table

1. **Read the constraints section** of the problem (look for "1 <= n <= ...")
2. **Find the matching row** in the table above
3. **Filter your technique options** — if n = 10^5, any O(n^2) approach will TLE (Time Limit Exceeded)
4. **Cross-reference** with the [[Problem Solving Flowchart]] to pick the exact technique

> [!example] Worked Example
> **Problem**: "Given an array of n integers, find the longest increasing subsequence."
> **Constraint**: n <= 2500
>
> **Analysis**:
> - n <= 2500 means O(n^2) is fine (2500^2 = 6.25M ops, well within limit)
> - Classic O(n^2) DP solution works
> - But if n <= 100,000, you'd need the O(n log n) solution using binary search + patience sorting

---

## 🔹 Common Constraint Patterns in Interviews

| Constraint Given | What They're Testing |
|---|---|
| n <= 10^4 to 10^5, array problems | O(n log n) or O(n) — sorting, hash maps, two pointers |
| String length <= 10^4 | O(n^2) DP is fine (e.g., longest common substring) |
| String length <= 10^6 | Need O(n) string algorithm (KMP, Z-algorithm, rolling hash) |
| Grid m x n, m,n <= 200 | O(m * n) BFS/DFS is fine (40K cells) |
| Grid m x n, m,n <= 1000 | O(m * n) still fine (10^6), but no O(m * n * k) |
| "Return all valid..." (n small) | Backtracking — they want you to enumerate |
| n <= 10^9 but answer is small | Binary search on the answer |
| Two constraints: n, m both given | Complexity often involves both, e.g., O(n * m) |

---

## 🔹 Edge Case Constraints

| Scenario | What It Signals |
|---|---|
| n = 1 | Watch for off-by-one / empty input handling |
| n = 0 or empty input | Always handle the empty case |
| Values up to 10^9 | Use `long` / `int64_t`, not `int` (overflow risk) |
| Values can be negative | Kadane's, two pointers, binary search boundaries all need adjustment |
| "Modulo 10^9 + 7" | Answer is huge — use modular arithmetic, likely DP or combinatorics |
| Multiple test cases (T up to 10^5) | Pre-computation or amortized O(1) per query |

> [!warning] Common Mistake
> Don't confuse **input size** with **value range**. If the problem says "array of n integers where each integer <= 10^9", the n might be only 10^5. Your complexity depends on n (the count), not the value range — unless you're doing counting sort or similar.

---

## 🔹 Quick Mental Math for Feasibility

```
10^8 operations ≈ 1 second (safe estimate)

n = 10^3  → n^2 = 10^6   ✅ (instant)
n = 10^4  → n^2 = 10^8   ⚠️ (borderline)
n = 10^5  → n^2 = 10^10  ❌ (TLE)
n = 10^5  → n log n ≈ 1.7 × 10^6  ✅ (fast)
n = 10^6  → n = 10^6     ✅ (fine for O(n))
n = 10^6  → n log n ≈ 2 × 10^7  ✅ (acceptable)
```

> [!tip] The "n^2 Cutoff" Rule
> **If n > 10,000, don't even try O(n^2).** This single rule eliminates many wrong approaches instantly.

---

## 🔹 Related

- [[Problem Solving Flowchart]] — after determining complexity, find the technique
- [[How to Pick the Right Algorithm]] — systematic technique selection
- [[How to Pick the Right Data Structure]] — data structure impacts complexity
