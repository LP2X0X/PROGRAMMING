---
tags:
  - algorithms
  - sorting
  - comparison
---

# Sorting Comparison

A master reference for comparing all sorting algorithms covered in this section. Use this to quickly decide which sort fits your situation.

---

## 🔹 Master Comparison Table

| Algorithm | Best | Average | Worst | Space | Stable? | In-Place? | When to Use |
|---|---|---|---|---|---|---|---|
| [[Simple Sorts#🔹 Bubble Sort\|Bubble Sort]] | O(n) | O(n^2) | O(n^2) | O(1) | Yes | Yes | Almost never; educational only |
| [[Simple Sorts#🔹 Selection Sort\|Selection Sort]] | O(n^2) | O(n^2) | O(n^2) | O(1) | No | Yes | When minimizing swaps matters (costly writes) |
| [[Simple Sorts#🔹 Insertion Sort\|Insertion Sort]] | O(n) | O(n^2) | O(n^2) | O(1) | Yes | Yes | Small arrays (n < 32); nearly sorted data |
| [[Merge Sort]] | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | No | Stability needed; linked lists; external sorting; guaranteed O(n log n) |
| [[Quick Sort]] | O(n log n) | O(n log n) | O(n^2) | O(log n) | No | Yes | General-purpose; best average performance on arrays |

---

## 🔹 Decision Flowchart

Use this to pick the right sort for a given situation:

```
START: What are you sorting?
  │
  ├─ Linked list?
  │    └─→ Merge Sort (O(1) extra space on linked lists)
  │
  ├─ Data too large for memory (external sort)?
  │    └─→ Merge Sort (k-way merge from disk)
  │
  ├─ Small array (n < ~32)?
  │    └─→ Insertion Sort (low overhead, fast on small n)
  │
  ├─ Nearly sorted data?
  │    └─→ Insertion Sort (O(n) best case)
  │
  ├─ Need stable sort?
  │    ├─ Can afford O(n) extra space?
  │    │    └─→ Merge Sort
  │    └─ Cannot afford extra space?
  │         └─→ Insertion Sort (if small n) or Timsort
  │
  ├─ Need guaranteed O(n log n) worst case?
  │    └─→ Merge Sort (or Heap Sort)
  │
  ├─ General-purpose, in-memory array?
  │    └─→ Quick Sort (with random/median-of-three pivot)
  │         Best cache performance, lowest constant factors
  │
  └─ Many duplicate keys?
       └─→ Quick Sort with 3-way partition (Dutch National Flag)
```

---

## 🔹 Visual Complexity Comparison

```
Time complexity growth (lower is better):

comparisons
    ^
    |                                           /  O(n^2)
    |                                        /     Bubble, Selection,
    |                                     /        Insertion (worst)
    |                                  /
    |                               /
    |                            /
    |                         /
    |                   ___/------- O(n log n)
    |               __/            Merge, Quick (avg)
    |           __/
    |       __/
    |    _/
    |  /__________________________ O(n)
    | /                            Insertion (best)
    +---------------------------------> n
```

---

## 🔹 Stability: Why It Matters

A **stable** sort preserves the relative order of elements with equal keys.

```
Example: Sort by grade, students already ordered by name

Input (sorted by name):
  Alice: B
  Bob:   A
  Carol: B
  Dave:  A

Stable sort by grade:        Unstable sort by grade:
  Bob:   A                     Dave:  A      ← Dave before Bob!
  Dave:  A                     Bob:   A
  Alice: B                     Carol: B      ← Carol before Alice!
  Carol: B                     Alice: B
```

Stability matters when you sort by multiple criteria sequentially (e.g., sort by name, then by grade — a stable sort on grade preserves the name ordering within each grade group).

**Stable:** Bubble Sort, Insertion Sort, Merge Sort
**Unstable:** Selection Sort, Quick Sort

---

## 🔹 What Standard Libraries Actually Use

Most production sort functions are **hybrid algorithms** that combine multiple sorts to get the best properties of each:

| Language/Library | Algorithm | Details |
|---|---|---|
| **Python** `sorted()` / `list.sort()` | **Timsort** | Merge Sort + Insertion Sort. Finds natural runs in the data and merges them. O(n log n) worst, O(n) best on nearly-sorted data. Stable. |
| **Java** `Arrays.sort()` (objects) | **Timsort** | Same as Python. Used for objects because stability matters. |
| **Java** `Arrays.sort()` (primitives) | **Dual-Pivot Quick Sort** | Quick Sort variant with two pivots. Not stable (fine for primitives — no identity). |
| **C++** `std::sort()` | **Introsort** | Quick Sort + Heap Sort fallback (if recursion exceeds 2 log n) + Insertion Sort (for n < 16). O(n log n) guaranteed. NOT stable. |
| **C++** `std::stable_sort()` | **Merge Sort** | Guarantees stability. O(n log n) time, O(n) space. |
| **C#/.NET** `Array.Sort()` | **Introsort** | Same hybrid as C++ std::sort. Quick Sort + Heap Sort + Insertion Sort. |
| **Go** `sort.Sort()` | **Pattern-Defeating Quicksort** | Quick Sort variant optimized for many real-world patterns. |

> [!note] The takeaway: in competitive programming, interviews, and understanding library behavior, you need to know [[Merge Sort]] and [[Quick Sort]]. The simple sorts matter as building blocks and for small-n base cases.

---

## 🔹 Lower Bound: Why O(n log n)?

Any **comparison-based** sorting algorithm must make at least O(n log n) comparisons in the worst case. This is a proven lower bound.

**Intuition:** There are n! possible orderings of n elements. Each comparison eliminates at most half the remaining possibilities (binary decision). You need at least log_2(n!) comparisons to distinguish all orderings. By Stirling's approximation, log_2(n!) = O(n log n).

This means Merge Sort and Quick Sort (average case) are **asymptotically optimal** among comparison-based sorts. To beat O(n log n), you need non-comparison sorts (Counting Sort, Radix Sort, Bucket Sort) that exploit special properties of the data.

---

## 🔹 Quick Reference: Which Sort When?

| Situation | Choose | Why |
|---|---|---|
| Interview: "sort this array" | Quick Sort or Merge Sort | Shows you know efficient algorithms |
| Interview: "sort a linked list" | Merge Sort | O(1) space, no random access needed |
| Small subarray (n < 32) | Insertion Sort | Lowest overhead |
| Data is almost sorted | Insertion Sort | O(n) best case |
| Must be stable | Merge Sort | Only O(n log n) stable comparison sort |
| Must be in-place | Quick Sort | O(log n) stack space only |
| Must guarantee O(n log n) | Merge Sort or Heap Sort | No degenerate cases |
| General-purpose on arrays | Quick Sort (randomized) | Best practical performance |
| External sorting (disk) | Merge Sort (k-way) | Sequential access, merges streams |
| Minimize writes/swaps | Selection Sort | Exactly n-1 swaps |

---

**See also:** [[Simple Sorts]] | [[Merge Sort]] | [[Quick Sort]]
