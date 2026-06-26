---
tags:
  - algorithms
  - sorting
  - divide-and-conquer
---

# Quick Sort

> **80/20 Algorithm** — Quick Sort is the most widely used sorting algorithm in practice. C++ `std::sort`, C `qsort`, and .NET `Array.Sort` all use Quick Sort variants. Understanding its partition scheme is critical — it also appears in problems like "find kth smallest element" (Quickselect).

**Real-world analogy:** Organizing a group of people by height. You pick one person as the "pivot" and tell everyone shorter to go left and everyone taller to go right. Then repeat the process for the left group and the right group independently.

---

## 🔹 Core Idea

1. **Pick a pivot** element from the array
2. **Partition** — rearrange elements so everything < pivot is on the left, everything > pivot is on the right, and the pivot is in its final sorted position
3. **Recurse** on the left and right subarrays (excluding the pivot — it's already placed)

Unlike [[Merge Sort]], the hard work happens **before** the recursion (partitioning), not after (merging).

---

## 🔹 The Partition Procedure (Lomuto Scheme)

This is the heart of Quick Sort. The Lomuto partition uses the **last element** as the pivot and maintains a boundary index `i` that tracks where the next "small" element should go.

### Step-by-Step Walkthrough

Input: **[10, 7, 8, 9, 1, 5]** — pivot = 5 (last element)

```
Initial state:
[10, 7, 8, 9, 1, 5]
  ^j                    i = -1 (nothing in "small" partition yet)
                         pivot = 5

--- j = 0: arr[0] = 10 ---
10 <= 5? NO → skip
[10, 7, 8, 9, 1, 5]     i = -1
  ^j

--- j = 1: arr[1] = 7 ---
7 <= 5? NO → skip
[10, 7, 8, 9, 1, 5]     i = -1
      ^j

--- j = 2: arr[2] = 8 ---
8 <= 5? NO → skip
[10, 7, 8, 9, 1, 5]     i = -1
         ^j

--- j = 3: arr[3] = 9 ---
9 <= 5? NO → skip
[10, 7, 8, 9, 1, 5]     i = -1
            ^j

--- j = 4: arr[4] = 1 ---
1 <= 5? YES → i++ (i = 0), swap arr[0] with arr[4]
[1, 7, 8, 9, 10, 5]     i = 0
 ^i          ^j

--- Done scanning. Place pivot: swap arr[i+1] with arr[high] ---
Swap arr[1] with arr[5]:
[1, 5, 8, 9, 10, 7]
    ^pivot in final position!

Partition result:
[1] [5] [8, 9, 10, 7]
 <5  =5     >5

Pivot index returned: 1
```

After partition, 5 is in its correct sorted position (index 1). Elements left of it are smaller, elements right are larger. Recurse on both sides.

### Full Recursion Tree

```
                      [10, 7, 8, 9, 1, 5]
                      pivot=5, partition at index 1
                     /          |          \
                   [1]         [5]     [8, 9, 10, 7]
                 (sorted)    (placed)   pivot=7, partition
                                       /     |        \
                                     []     [7]   [9, 10, 8]
                                           (placed) pivot=8
                                                  /    |     \
                                                []    [8]  [10, 9]
                                                    (placed) pivot=9
                                                            /   |  \
                                                          []   [9] [10]

Result: [1, 5, 7, 8, 9, 10]
```

---

## 🔹 Pivot Selection Strategies

The pivot choice determines whether Quick Sort hits O(n log n) or degrades to O(n^2).

| Strategy | How | Pros | Cons |
|---|---|---|---|
| **Last element** | `pivot = arr[high]` | Simplest to implement | O(n^2) on sorted/reverse-sorted input |
| **First element** | `pivot = arr[low]` | Simple | Same problem on sorted input |
| **Random** | `pivot = arr[rand(low, high)]` | Avoids adversarial worst cases | Needs RNG; still O(n^2) possible but astronomically unlikely |
| **Median-of-three** | Median of `arr[low]`, `arr[mid]`, `arr[high]` | Good pivot in practice; handles sorted input well | Slightly more code; still O(n^2) possible with crafted input |

> [!tip] **In practice, use median-of-three or random pivot.** The Lomuto scheme with last-element pivot is for learning; production implementations avoid it.

---

## 🔹 Complexity Analysis

| Case | Time | When |
|---|---|---|
| **Best** | O(n log n) | Pivot always splits array roughly in half |
| **Average** | O(n log n) | Random pivot selection |
| **Worst** | O(n^2) | Already sorted + always picking min/max as pivot |

**Space:** O(log n) average for the recursion stack. O(n) worst case (every partition is maximally unbalanced → recursion depth = n).

**Stable?** No — the partition swaps distant elements, disrupting relative order of equal values.

**In-place?** Yes — partitioning is done within the original array (no temp arrays needed).

### Why O(n^2) on Sorted Input (with Lomuto/last-element pivot)

```
Sorted input: [1, 2, 3, 4, 5]
Pivot = 5 (last element)
Partition: [1, 2, 3, 4] [5] []     ← maximally unbalanced!
           └── all n-1 elements on one side

Next: pivot = 4
Partition: [1, 2, 3] [4] []        ← still unbalanced

This gives n + (n-1) + (n-2) + ... + 1 = O(n^2) comparisons
```

The fix: **random pivot** or **median-of-three** makes this degenerate case virtually impossible.

---

## 🔹 Why Quick Sort Is Preferred in Practice

Despite Merge Sort's guaranteed O(n log n), Quick Sort is often faster in practice:

1. **Cache-friendly:** Works in-place on contiguous memory. Sequential access patterns play well with CPU caches. Merge Sort copies data to temporary arrays, causing cache misses
2. **No extra memory:** O(log n) stack space vs. O(n) temporary array for Merge Sort
3. **Small constant factors:** The inner loop is a simple comparison + conditional swap — very tight machine code
4. **Tail-call optimization:** Recurse on the smaller partition first, use a loop for the larger one — keeps stack depth O(log n) even in worst case

> Most standard library sort functions use **Introsort**: Quick Sort that switches to Heap Sort if recursion depth exceeds 2 log n (guaranteeing O(n log n) worst case), and switches to Insertion Sort for small partitions (n < 16).

---

## 🔹 Complete C++ Implementation

```cpp
// Lomuto partition: uses last element as pivot
int partition(int arr[], int low, int high) {
    int pivot = arr[high];
    int i = low - 1;  // boundary of "small" elements

    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i + 1], arr[high]);  // place pivot in correct position
    return i + 1;                 // return pivot's final index
}

void quickSort(int arr[], int low, int high) {
    if (low >= high) return;

    int pivotIdx = partition(arr, low, high);
    quickSort(arr, low, pivotIdx - 1);   // sort left of pivot
    quickSort(arr, pivotIdx + 1, high);  // sort right of pivot
}

// Usage:
// int arr[] = {10, 7, 8, 9, 1, 5};
// int n = sizeof(arr) / sizeof(arr[0]);
// quickSort(arr, 0, n - 1);
```

### With Random Pivot (Recommended)

```cpp
int randomPartition(int arr[], int low, int high) {
    int randIdx = low + rand() % (high - low + 1);
    swap(arr[randIdx], arr[high]);  // move random pivot to end
    return partition(arr, low, high);  // use standard Lomuto
}

void quickSort(int arr[], int low, int high) {
    if (low >= high) return;

    int pivotIdx = randomPartition(arr, low, high);
    quickSort(arr, low, pivotIdx - 1);
    quickSort(arr, pivotIdx + 1, high);
}
```

---

## 🔹 Quick Sort vs. Merge Sort

| Aspect | Quick Sort | [[Merge Sort]] |
|---|---|---|
| Average time | O(n log n) | O(n log n) |
| Worst time | O(n^2) | O(n log n) |
| Space | O(log n) | O(n) |
| Stable | No | Yes |
| In-place | Yes | No |
| Cache behavior | Excellent | Poor (copies to temp arrays) |
| Linked lists | Poor (no random access) | Excellent (O(1) space merge) |
| Real-world speed | Faster (better constants) | Slower (memory allocation) |

---

## 🔹 Common Pitfalls

1. **Infinite recursion:** If partition returns `low` or `high` without excluding the pivot, recursion never terminates. Always recurse on `[low, pivotIdx-1]` and `[pivotIdx+1, high]`
2. **O(n^2) on sorted input:** Using first/last element pivot on sorted arrays. Fix: use random or median-of-three
3. **Stack overflow on large arrays:** Deep recursion on unbalanced partitions. Fix: recurse on the smaller partition first, iterate on the larger one (tail-call elimination)
4. **`<=` in partition comparison:** Lomuto uses `arr[j] <= pivot` (not strict `<`). Using `<` causes all elements equal to the pivot to end up on the right, creating unbalanced partitions when there are many duplicates
5. **Not seeding random:** Call `srand(time(0))` before using random pivot

---

## 🔹 Related Algorithms

- **Quickselect:** Uses partition to find the kth smallest element in O(n) average time — no need to fully sort
- **3-way partition (Dutch National Flag):** Handles many duplicate keys efficiently — partitions into `< pivot`, `= pivot`, `> pivot`
- **Introsort:** Quick Sort + Heap Sort fallback + Insertion Sort for small arrays — used by C++ `std::sort`

---

**See also:** [[Merge Sort]] | [[Sorting Comparison]] | [[Simple Sorts]]
