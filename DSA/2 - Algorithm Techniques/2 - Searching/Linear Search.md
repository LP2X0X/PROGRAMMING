---
tags:
  - algorithms
  - searching
  - fundamental
---

## 🔹 Real-World Analogy

Imagine you are looking for a specific book on a bookshelf that has no particular order. You have no choice but to start at one end and check every single book, one by one, until you find the one you want or reach the end of the shelf. That is linear search.

## 🔹 Definition

Linear search (also called sequential search) scans each element of a collection one at a time, from beginning to end, comparing each element against the target value. It returns the index of the first match, or `-1` if the target is not found.

## 🔹 How It Works — Step by Step

**Goal:** Find value `7` in array `[3, 8, 1, 7, 5, 2]`

```
Index:   0    1    2    3    4    5
Array: [ 3 ][ 8 ][ 1 ][ 7 ][ 5 ][ 2 ]

Step 1:  ^
         i=0, arr[0]=3, 3 == 7? No  --> move right

Step 2:       ^
         i=1, arr[1]=8, 8 == 7? No  --> move right

Step 3:            ^
         i=2, arr[2]=1, 1 == 7? No  --> move right

Step 4:                 ^
         i=3, arr[3]=7, 7 == 7? YES --> return 3
```

**Result:** Found at index `3` after checking 4 elements.

If the target were `9` (not in the array), we would check all 6 elements and return `-1`.

## 🔹 Complexity Analysis

| Case    | Time   | Space |
| ------- | ------ | ----- |
| Best    | O(1)   | O(1)  |
| Average | O(n)   | O(1)  |
| Worst   | O(n)   | O(1)  |

- Best case: target is the first element.
- Worst case: target is the last element or not present at all.

## 🔹 When to Use

- **Unsorted data** — you cannot use [[Binary Search]] without sorting first, and sorting costs O(n log n). If you only search once, linear search is cheaper.
- **Small arrays** — for arrays smaller than ~64 elements, the overhead of binary search setup is not worth it. Linear search with cache-friendly sequential access can be faster in practice.
- **Linked lists** — random access is O(n) in a linked list, so binary search gives no benefit. Linear search is the natural fit.
- **Single-pass searches** — when data arrives as a stream and you process it once.
- **Searching by a non-key field** — when you need to find an element by a property that is not indexed or sorted.

## 🔹 Template Code (C++)

```cpp
// Returns the index of target in arr, or -1 if not found.
int linearSearch(int arr[], int n, int target) {
    for (int i = 0; i < n; i++) {
        if (arr[i] == target) {
            return i;
        }
    }
    return -1;
}
```

**Generic version with templates:**

```cpp
template <typename T>
int linearSearch(const T arr[], int n, const T& target) {
    for (int i = 0; i < n; i++) {
        if (arr[i] == target) {
            return i;
        }
    }
    return -1;
}
```

## 🔹 Common Pitfalls

1. **Forgetting to handle "not found"** — always return a sentinel value (`-1`) or use a boolean flag. Do not assume the target is always present.
2. **Off-by-one on loop bound** — use `i < n`, not `i <= n`. The last valid index is `n - 1`.
3. **Returning too early in "find all" variants** — if you need all occurrences, do not return on the first match. Collect results instead.

## 🔹 When [[Binary Search]] Is Better (and When It Is Not)

| Scenario                                  | Winner         | Why                                             |
| ----------------------------------------- | -------------- | ----------------------------------------------- |
| Sorted array, many searches               | Binary Search  | O(log n) per search amortizes the sort cost     |
| Unsorted array, single search             | Linear Search  | Sorting first costs O(n log n), worse than O(n) |
| Very small array (< ~64 elements)         | Linear Search  | Less overhead, better cache behavior            |
| Linked list                               | Linear Search  | No random access, binary search cannot help     |
| Sorted array, single search               | Binary Search  | O(log n) < O(n), no sort cost needed            |
| Searching by non-comparable property      | Linear Search  | Binary search requires a total ordering         |
