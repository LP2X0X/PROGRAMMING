---
tags:
  - algorithms
  - sorting
  - divide-and-conquer
---

# Merge Sort

> **80/20 Algorithm** — Merge Sort is one of the most important sorting algorithms to deeply understand. Its divide-and-conquer pattern recurs throughout algorithm design, and it guarantees O(n log n) in all cases.

**Real-world analogy:** Imagine sorting a huge pile of exams. You split the pile in half, give each half to a helper, they split again, and so on until each person holds one exam. Then pairs of people merge their sorted stacks together by comparing the top sheets. The merged stacks flow upward until the entire pile is sorted.

---

## 🔹 Core Idea

1. **Divide** — Split the array in half
2. **Conquer** — Recursively sort each half (base case: single element is already sorted)
3. **Combine** — Merge the two sorted halves into one sorted array using a two-pointer technique

This is the canonical example of **divide and conquer**.

---

## 🔹 Full Walkthrough

Input array: **[38, 27, 43, 3, 9, 82, 10]**

### Splitting Phase (Top-Down)

```
                    [38, 27, 43, 3, 9, 82, 10]
                   /                           \
          [38, 27, 43, 3]                [9, 82, 10]
          /             \                /          \
      [38, 27]       [43, 3]        [9, 82]       [10]
      /      \       /     \        /     \          |
    [38]    [27]   [43]    [3]    [9]    [82]      [10]
```

Each leaf is a single element — trivially sorted.

### Merging Phase (Bottom-Up)

```
    [38]    [27]   [43]    [3]    [9]    [82]      [10]
      \      /       \     /        \     /          |
      [27, 38]       [3, 43]       [9, 82]        [10]
          \             /                \          /
       [3, 27, 38, 43]                [9, 10, 82]
                   \                       /
            [3, 9, 10, 27, 38, 43, 82]
```

---

## 🔹 The Merge Procedure (Detail)

This is the heart of the algorithm. Given two sorted arrays, produce one sorted array.

**Merging [27, 38] and [3, 43]:**

```
Left:  [27, 38]    Right: [3, 43]    Result: []
        ^L                 ^R

Step 1: Compare L=27 vs R=3  → 3 < 27  → take 3
Left:  [27, 38]    Right: [3, 43]    Result: [3]
        ^L                    ^R

Step 2: Compare L=27 vs R=43 → 27 < 43 → take 27
Left:  [27, 38]    Right: [3, 43]    Result: [3, 27]
            ^L                ^R

Step 3: Compare L=38 vs R=43 → 38 < 43 → take 38
Left:  [27, 38]    Right: [3, 43]    Result: [3, 27, 38]
               ^L(done)      ^R

Step 4: Left exhausted → copy remaining right
                       Right: [3, 43]    Result: [3, 27, 38, 43]
                                  ^R(done)
```

**Merging [3, 27, 38, 43] and [9, 10, 82]:**

```
Left:  [3, 27, 38, 43]    Right: [9, 10, 82]    Result: []
        ^L                         ^R

Step 1: 3 < 9   → take 3      Result: [3]
Step 2: 27 > 9  → take 9      Result: [3, 9]
Step 3: 27 > 10 → take 10     Result: [3, 9, 10]
Step 4: 27 < 82 → take 27     Result: [3, 9, 10, 27]
Step 5: 38 < 82 → take 38     Result: [3, 9, 10, 27, 38]
Step 6: 43 < 82 → take 43     Result: [3, 9, 10, 27, 38, 43]
Step 7: Left done → copy 82   Result: [3, 9, 10, 27, 38, 43, 82]
```

---

## 🔹 Recurrence Relation

```
T(n) = 2T(n/2) + O(n)
       ─────────   ───
       two halves   merge cost
```

By the **Master Theorem** (case 2: a = 2, b = 2, f(n) = n, log_b(a) = 1):

**T(n) = O(n log n)**

This holds for best, average, AND worst case — the splitting and merging pattern is the same regardless of input order.

---

## 🔹 Complexity Analysis

| Case | Time | Why |
|---|---|---|
| Best | O(n log n) | Always splits and merges the same way |
| Average | O(n log n) | Same structure regardless of input |
| Worst | O(n log n) | No degenerate cases possible |
| Space | O(n) | Temporary array needed for merging |

- **Stable:** Yes — when equal elements are compared during merge, the left element is taken first, preserving original order
- **In-place:** No — requires O(n) extra space for the temporary merge array
- **Not adaptive:** Does the same amount of work on sorted and random input

---

## 🔹 Complete C++ Implementation

```cpp
// Merge two sorted subarrays: arr[left..mid] and arr[mid+1..right]
void merge(int arr[], int left, int mid, int right) {
    int n1 = mid - left + 1;   // size of left half
    int n2 = right - mid;      // size of right half

    // Create temporary arrays
    vector<int> L(n1), R(n2);

    for (int i = 0; i < n1; i++) L[i] = arr[left + i];
    for (int j = 0; j < n2; j++) R[j] = arr[mid + 1 + j];

    // Two-pointer merge
    int i = 0, j = 0, k = left;

    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) {     // <= ensures stability
            arr[k] = L[i];
            i++;
        } else {
            arr[k] = R[j];
            j++;
        }
        k++;
    }

    // Copy remaining elements (only one of these loops will execute)
    while (i < n1) { arr[k] = L[i]; i++; k++; }
    while (j < n2) { arr[k] = R[j]; j++; k++; }
}

void mergeSort(int arr[], int left, int right) {
    if (left >= right) return;  // base case: 0 or 1 elements

    int mid = left + (right - left) / 2;  // avoids overflow vs (left+right)/2

    mergeSort(arr, left, mid);       // sort left half
    mergeSort(arr, mid + 1, right);  // sort right half
    merge(arr, left, mid, right);    // merge sorted halves
}

// Usage:
// int arr[] = {38, 27, 43, 3, 9, 82, 10};
// int n = sizeof(arr) / sizeof(arr[0]);
// mergeSort(arr, 0, n - 1);
```

---

## 🔹 When to Use Merge Sort

| Scenario | Why Merge Sort Wins |
|---|---|
| **Linked lists** | Merging linked lists requires no extra space (just pointer rewiring). Merge Sort is O(n log n) with O(1) extra space on linked lists — strictly better than [[Quick Sort]] here. |
| **Stability required** | Merge Sort is stable; [[Quick Sort]] is not. Use when you need to preserve the relative order of equal elements. |
| **External sorting** | Sorting data that doesn't fit in memory (e.g., sorting a 100 GB file). Split into chunks that fit in RAM, sort each chunk, then merge chunks from disk. |
| **Guaranteed O(n log n)** | No worst-case degradation. If you cannot tolerate occasional O(n^2) and don't want to implement Introsort, Merge Sort is the safe choice. |
| **Parallelism** | The two recursive calls are independent — trivially parallelizable. |

---

## 🔹 When NOT to Use Merge Sort

- **Memory-constrained:** Requires O(n) extra space on arrays (unlike [[Quick Sort]] which is in-place)
- **Small arrays:** Overhead of recursion and copying. Use [[Simple Sorts|Insertion Sort]] for n < ~32
- **Cache performance on arrays:** [[Quick Sort]] has better locality of reference since it works in-place

---

## 🔹 Common Pitfalls

1. **Overflow in midpoint calculation:** Use `mid = left + (right - left) / 2` instead of `(left + right) / 2`
2. **Stability bug:** In the merge step, use `<=` (not `<`) when comparing `L[i]` with `R[j]` to preserve stability
3. **Off-by-one in merge bounds:** `left..mid` is the left half, `mid+1..right` is the right half. Getting this wrong causes infinite recursion or missed elements
4. **Forgetting to copy remainders:** After the main merge loop, one subarray still has elements — you must copy them

---

## 🔹 Variations

- **Bottom-up Merge Sort:** Iterative version that avoids recursion. Start by merging pairs of size 1, then 2, then 4, etc. Same time complexity, avoids stack overhead
- **Natural Merge Sort:** Identifies existing sorted runs in the input and merges them. This is the basis of **Timsort** (Python, Java)
- **k-way Merge:** Merge k sorted arrays using a min-heap — used in external sorting and merge phase of MapReduce

---

**See also:** [[Quick Sort]] | [[Sorting Comparison]] | [[Simple Sorts]]
