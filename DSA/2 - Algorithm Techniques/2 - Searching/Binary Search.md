---
tags:
  - algorithms
  - searching
  - fundamental
---

## 🔹 Real-World Analogy

Think of looking up a word in a physical dictionary. You do not start at page 1 and read every entry. Instead, you open it roughly to the middle. If your word comes alphabetically before the page you opened to, you flip to the left half. If it comes after, you flip to the right half. You keep halving until you land on the exact page. Each decision eliminates half of the remaining pages. That is binary search.

## 🔹 Definition

Binary search is a divide-and-conquer algorithm that finds a target value within a **sorted** array. It maintains two boundaries (`left` and `right`), computes the midpoint, compares the midpoint value to the target, and discards the half that cannot contain the target. This repeats until the target is found or the search space is empty.

> **Prerequisite:** The array MUST be sorted. Binary search on an unsorted array produces garbage results.

## 🔹 How It Works — Step by Step

**Goal:** Find value `23` in sorted array `[2, 5, 8, 12, 16, 23, 38, 56, 72, 91]`

```
Index:   0    1    2    3    4    5    6    7    8    9
Array: [ 2 ][ 5 ][ 8 ][12 ][16 ][23 ][38 ][56 ][72 ][91 ]

Step 1: left=0, right=9
        mid = 0 + (9-0)/2 = 4
        arr[4] = 16
        16 < 23 --> target is in RIGHT half --> left = mid + 1 = 5

         0    1    2    3    4    5    6    7    8    9
       [ 2 ][ 5 ][ 8 ][12 ][16 ][23 ][38 ][56 ][72 ][91 ]
                                  L              ^         R
                                       search this half -->

Step 2: left=5, right=9
        mid = 5 + (9-5)/2 = 7
        arr[7] = 56
        56 > 23 --> target is in LEFT half --> right = mid - 1 = 6

         0    1    2    3    4    5    6    7    8    9
       [ 2 ][ 5 ][ 8 ][12 ][16 ][23 ][38 ][56 ][72 ][91 ]
                                  L         R
                                  <-- search this half

Step 3: left=5, right=6
        mid = 5 + (6-5)/2 = 5
        arr[5] = 23
        23 == 23 --> FOUND! Return index 5
```

**Result:** Found at index `5` after only 3 comparisons (vs. 6 for [[Linear Search]]).

### Walkthrough — Target NOT Found

**Goal:** Find value `20` in the same array.

```
Index:   0    1    2    3    4    5    6    7    8    9
Array: [ 2 ][ 5 ][ 8 ][12 ][16 ][23 ][38 ][56 ][72 ][91 ]

Step 1: left=0, right=9, mid=4, arr[4]=16
        16 < 20 --> left = 5

Step 2: left=5, right=9, mid=7, arr[7]=56
        56 > 20 --> right = 6

Step 3: left=5, right=6, mid=5, arr[5]=23
        23 > 20 --> right = 4

Step 4: left=5, right=4
        left > right --> SEARCH SPACE EMPTY --> return -1
```

**Result:** Not found. Only 3 comparisons to rule out a 10-element array.

## 🔹 Why O(log n)?

Each step cuts the search space in half. Starting with `n` elements:

```
After step 1:  n/2   elements remain
After step 2:  n/4   elements remain
After step 3:  n/8   elements remain
...
After step k:  n/2^k elements remain

Search ends when n/2^k = 1  -->  k = log2(n)
```

For `n = 1,000,000` elements, binary search needs at most **20** comparisons. Linear search needs up to 1,000,000.

## 🔹 Complexity Analysis

| Variant   | Time      | Space |
| --------- | --------- | ----- |
| Iterative | O(log n)  | O(1)  |
| Recursive | O(log n)  | O(log n) due to call stack |

Always prefer the iterative version in practice to avoid stack overflow on large arrays.

## 🔹 Standard Binary Search — Iterative (C++)

```cpp
// Returns the index of target in sorted arr, or -1 if not found.
int binarySearch(int arr[], int n, int target) {
    int left = 0;
    int right = n - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;  // avoids overflow

        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return -1;  // not found
}
```

## 🔹 Standard Binary Search — Recursive (C++)

```cpp
int binarySearchRecursive(int arr[], int left, int right, int target) {
    if (left > right) {
        return -1;  // base case: not found
    }

    int mid = left + (right - left) / 2;

    if (arr[mid] == target) {
        return mid;
    } else if (arr[mid] < target) {
        return binarySearchRecursive(arr, mid + 1, right, target);
    } else {
        return binarySearchRecursive(arr, left, mid - 1, target);
    }
}

// Usage: binarySearchRecursive(arr, 0, n - 1, target);
```

## 🔹 Common Pitfalls

### Pitfall 1: Integer Overflow in Mid Calculation

```cpp
// WRONG — overflows when left + right > INT_MAX
int mid = (left + right) / 2;

// CORRECT — always safe
int mid = left + (right - left) / 2;
```

If `left = 2,000,000,000` and `right = 2,000,000,000`, then `left + right = 4,000,000,000` which overflows a 32-bit signed integer. The safe formula avoids this entirely.

### Pitfall 2: Off-by-One Errors (Inclusive vs. Exclusive Bounds)

There are two common conventions:

| Convention         | Init              | Loop condition   | Narrow left     | Narrow right    |
| ------------------ | ----------------- | ---------------- | --------------- | --------------- |
| **Both inclusive**  | `left=0, right=n-1` | `left <= right` | `left = mid+1`  | `right = mid-1` |
| **Right exclusive** | `left=0, right=n`   | `left < right`  | `left = mid+1`  | `right = mid`   |

Pick ONE convention and stick with it. Mixing them is the #1 source of infinite loops and off-by-one bugs. This note uses the **both inclusive** convention throughout.

### Pitfall 3: Forgetting the Sorted Prerequisite

Binary search silently produces wrong answers on unsorted data. Always verify the input is sorted, or sort it first (O(n log n) one-time cost).

### Pitfall 4: Infinite Loop

If you write `right = mid` in the inclusive convention (instead of `right = mid - 1`), the loop can get stuck when `left == right == mid`. Always shrink the search space by at least one element each step.

---

## 🔹 Variation 1: Find First Occurrence (Lower Bound)

When the array has duplicates, standard binary search returns *any* matching index. This variation finds the **leftmost** (first) occurrence.

### ASCII Walkthrough

**Goal:** Find first occurrence of `5` in `[1, 3, 5, 5, 5, 8, 9]`

```
Index:   0    1    2    3    4    5    6
Array: [ 1 ][ 3 ][ 5 ][ 5 ][ 5 ][ 8 ][ 9 ]

Step 1: left=0, right=6, mid=3, arr[3]=5
        5 == target --> found a match, but is it the FIRST?
        Save result=3, keep searching LEFT: right = mid - 1 = 2

Step 2: left=0, right=2, mid=1, arr[1]=3
        3 < 5 --> left = 2

Step 3: left=2, right=2, mid=2, arr[2]=5
        5 == target --> found earlier match!
        Save result=2, right = mid - 1 = 1

Step 4: left=2, right=1 --> left > right --> stop

Result: First occurrence at index 2.
```

### Template Code

```cpp
// Returns index of the first element == target, or -1 if not found.
int lowerBound(int arr[], int n, int target) {
    int left = 0, right = n - 1;
    int result = -1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (arr[mid] == target) {
            result = mid;        // record this match
            right = mid - 1;     // keep searching left for earlier match
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return result;
}
```

## 🔹 Variation 2: Find Last Occurrence (Upper Bound)

Same idea, but when we find a match, we search **right** for a later occurrence.

### Template Code

```cpp
// Returns index of the last element == target, or -1 if not found.
int upperBound(int arr[], int n, int target) {
    int left = 0, right = n - 1;
    int result = -1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (arr[mid] == target) {
            result = mid;        // record this match
            left = mid + 1;      // keep searching right for later match
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return result;
}
```

**Counting occurrences:** `count = upperBound(...) - lowerBound(...) + 1` (if both return != -1).

## 🔹 Variation 3: Find Insertion Point

Find the index where `target` should be inserted to keep the array sorted. This is equivalent to C++'s `std::lower_bound`.

```cpp
// Returns the index where target should be inserted.
// If target exists, returns the index of its first occurrence.
int insertionPoint(int arr[], int n, int target) {
    int left = 0, right = n;  // note: right = n (exclusive)

    while (left < right) {
        int mid = left + (right - left) / 2;

        if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid;       // arr[mid] >= target
        }
    }

    return left;  // insertion point
}
```

## 🔹 Variation 4: Search in Rotated Sorted Array

A sorted array that has been rotated (e.g., `[4, 5, 6, 7, 0, 1, 2]` — originally `[0, 1, 2, 4, 5, 6, 7]` rotated at index 4).

**Key insight:** At least one half (left or right of `mid`) is always sorted. Determine which half is sorted, then check if the target falls within that sorted half.

```
Original:  [0, 1, 2, 4, 5, 6, 7]
Rotated:   [4, 5, 6, 7, 0, 1, 2]
                       ^
                    pivot point
```

```cpp
int searchRotated(int arr[], int n, int target) {
    int left = 0, right = n - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (arr[mid] == target) return mid;

        // Left half is sorted
        if (arr[left] <= arr[mid]) {
            if (arr[left] <= target && target < arr[mid]) {
                right = mid - 1;  // target is in sorted left half
            } else {
                left = mid + 1;   // target is in right half
            }
        }
        // Right half is sorted
        else {
            if (arr[mid] < target && target <= arr[right]) {
                left = mid + 1;   // target is in sorted right half
            } else {
                right = mid - 1;  // target is in left half
            }
        }
    }

    return -1;
}
```

---

## 🔹 The "Binary Search on Answer" Pattern

This is arguably the most powerful application of binary search in competitive programming and interviews. Instead of searching for a target in an array, you binary search over a **range of possible answers** to an optimization problem.

### When to Recognize This Pattern

You can use binary search on the answer when:
1. The problem asks to **minimize the maximum** or **maximize the minimum** of something.
2. There is a **monotonic relationship**: if answer `x` is feasible, then all answers `> x` (or `< x`) are also feasible.
3. You can write a `feasible(x)` function that checks whether a given answer `x` is achievable.

### The Template

```
                       infeasible | feasible
Answer space: [lo .................|.......... hi]
                                  ^
                          find this boundary
```

```cpp
// Binary search on the answer: find the minimum feasible value.
// feasible(x) returns true if answer x is achievable.
int binarySearchOnAnswer(int lo, int hi) {
    int result = hi;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (feasible(mid)) {
            result = mid;       // mid works, try smaller
            hi = mid - 1;
        } else {
            lo = mid + 1;       // mid too small, try bigger
        }
    }

    return result;
}
```

### Example: Minimum Capacity to Ship Packages in D Days

**Problem:** Given an array `weights[]` and a deadline `D` days, find the minimum ship capacity such that all packages can be shipped within `D` days. Packages must be shipped in order.

```
weights = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],  D = 5

Possible capacities: from max(weights)=10  to  sum(weights)=55
                     (must carry the heaviest single package at minimum)

Binary search the answer in range [10, 55]:
  mid = 32 --> feasible? Need 2 days. Yes. Try smaller.
  mid = 21 --> feasible? Need 3 days. Yes. Try smaller.
  mid = 15 --> feasible? Need 5 days. Yes. Try smaller.
  mid = 12 --> feasible? Need 6 days. No.  Try bigger.
  mid = 13 --> feasible? Need 6 days. No.  Try bigger.
  mid = 14 --> feasible? Need 5 days. Yes. Try smaller.
  Answer: 15
```

```cpp
bool canShipInDDays(int weights[], int n, int capacity, int D) {
    int days = 1;
    int currentLoad = 0;

    for (int i = 0; i < n; i++) {
        if (currentLoad + weights[i] > capacity) {
            days++;
            currentLoad = 0;
        }
        currentLoad += weights[i];
    }

    return days <= D;
}

int shipWithinDays(int weights[], int n, int D) {
    int lo = *max_element(weights, weights + n);  // can't be less than heaviest
    int hi = accumulate(weights, weights + n, 0);  // ship everything in 1 day
    int result = hi;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (canShipInDDays(weights, n, mid, D)) {
            result = mid;
            hi = mid - 1;
        } else {
            lo = mid + 1;
        }
    }

    return result;
}
```

### More "Binary Search on Answer" Problems

| Problem                                        | Search space             | Feasibility check                     |
| ----------------------------------------------- | ------------------------ | ------------------------------------- |
| Minimum capacity to ship in D days              | [max(w), sum(w)]         | Can all packages ship in <= D days?   |
| Split array into M subarrays, minimize max sum  | [max(arr), sum(arr)]     | Can we split into <= M parts?         |
| Koko eating bananas (min speed)                 | [1, max(piles)]          | Can she finish in <= H hours?         |
| Aggressive cows (maximize minimum distance)     | [1, max_pos - min_pos]   | Can we place all cows with gap >= d?  |
| Painter's partition                             | [max(boards), sum(boards)] | Can k painters finish in time <= t?   |

---

## 🔹 Quick Reference Card

| Operation               | Convention     | Key line                          |
| ------------------------ | -------------- | --------------------------------- |
| Standard search          | Inclusive      | `left <= right`, return mid       |
| First occurrence         | Inclusive      | On match: `result=mid, right=mid-1` |
| Last occurrence          | Inclusive      | On match: `result=mid, left=mid+1`  |
| Insertion point          | Right-exclusive | `left < right`, `right=mid`       |
| Search on answer (min)   | Inclusive      | Feasible: `result=mid, hi=mid-1`  |
| Search on answer (max)   | Inclusive      | Feasible: `result=mid, lo=mid+1`  |

## 🔹 See Also

- [[Linear Search]] — simpler alternative when data is unsorted or small
- [[Two Pointers Technique]] — another O(n) technique often used on sorted arrays alongside binary search
