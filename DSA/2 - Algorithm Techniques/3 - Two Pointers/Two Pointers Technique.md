---
tags:
  - algorithms
  - technique
  - two-pointers
---

## 🔹 Real-World Analogy

Imagine two people searching a **sorted bookshelf** for a specific pair of books whose combined page count equals a target number. One person starts at the **left end** (smallest books), the other at the **right end** (biggest books). If their combined page count is too small, the left person moves right toward bigger books. If it's too big, the right person moves left toward smaller books. They converge toward the answer without either person needing to scan the entire shelf.

This is the essence of two pointers: **two references moving through data according to rules**, eliminating large portions of the search space at each step.

---

## 🔹 What Is Two Pointers?

Two pointers is a technique where you maintain **two index variables** that traverse a data structure (usually a sorted array, string, or linked list) according to specific movement rules.

**Key insight**: Instead of checking every possible pair with nested loops — O(n^2) — you use the structure of the data (sorted order, duplicates, etc.) to **move each pointer at most n times total**, achieving **O(n)** time.

It is one of the highest-leverage algorithm techniques: simple to implement, easy to reason about, and appears in a massive number of interview and competitive programming problems.

---

## 🔹 Three Main Patterns

### Pattern 1: Opposite-Direction Pointers

Both pointers start at **opposite ends** of the array and **move toward each other** until they meet.

**When to use**: The array is sorted and you need to find a pair that satisfies some condition (target sum, palindrome check, etc.).

#### ASCII Walkthrough — Two Sum in Sorted Array

**Problem**: Find two numbers in `[1, 2, 4, 6, 7, 11]` that sum to `9`.

```
Step 1: L points to 1, R points to 11
         sum = 1 + 11 = 12 > 9  →  move R left (sum too big)

         [1,  2,  4,  6,  7, 11]
          L                   R
          ^                   ^
          sum = 12 > 9 → R moves left


Step 2: L points to 1, R points to 7
         sum = 1 + 7 = 8 < 9  →  move L right (sum too small)

         [1,  2,  4,  6,  7, 11]
          L               R
          ^               ^
          sum = 8 < 9 → L moves right


Step 3: L points to 2, R points to 7
         sum = 2 + 7 = 9 == 9  →  FOUND!

         [1,  2,  4,  6,  7, 11]
              L           R
              ^           ^
              sum = 9 == 9 → FOUND! return {L, R}
```

**Why it works**: Because the array is sorted:
- If `sum < target`, the only way to increase it is to move `L` right (bigger value)
- If `sum > target`, the only way to decrease it is to move `R` left (smaller value)
- Each step eliminates an entire row/column of the pair matrix

#### Other Classic Problems

| Problem | Key Idea |
|---|---|
| **Palindrome check** | L and R compare characters moving inward |
| **Container with most water** | Move the pointer with the shorter height |
| **3Sum** | Fix one element, run two-pointer on the rest |
| **Trapping rain water** | Track maxLeft and maxRight with two pointers |

#### Template Code — Opposite Direction

```cpp
// Two Sum in sorted array — find pair summing to target
// Returns indices of the two elements, or {-1, -1} if not found
pair<int,int> twoSumSorted(vector<int>& arr, int target) {
    int left = 0;
    int right = arr.size() - 1;

    while (left < right) {
        int sum = arr[left] + arr[right];

        if (sum == target) {
            return {left, right};
        } else if (sum < target) {
            left++;     // need bigger sum → move left pointer right
        } else {
            right--;    // need smaller sum → move right pointer left
        }
    }
    return {-1, -1};   // no pair found
}
```

```
General pattern:

    int left = 0, right = n - 1;
    while (left < right) {
        // compute something with arr[left] and arr[right]
        // decide which pointer to move based on comparison
    }
```

---

### Pattern 2: Same-Direction Pointers (Fast / Slow)

Both pointers start at the **same end** and move in the **same direction**, but at **different speeds** or under different conditions.

- **Slow pointer** = marks the "write" position or the "answer so far"
- **Fast pointer** = scans ahead looking for the next useful element

**When to use**: In-place modifications, cycle detection, finding middle element.

#### ASCII Walkthrough — Remove Duplicates In-Place

**Problem**: Remove duplicates from sorted array `[1, 1, 2, 2, 3, 4, 4]`, return new length.

```
Initial state:
         [1,  1,  2,  2,  3,  4,  4]
          S   F
          slow=0, fast=1

Step 1:  arr[F]=1 == arr[S]=1  →  duplicate, skip
         [1,  1,  2,  2,  3,  4,  4]
          S       F
          slow=0, fast=2

Step 2:  arr[F]=2 != arr[S]=1  →  new value! S++, copy arr[F] to arr[S]
         [1,  2,  2,  2,  3,  4,  4]
              S       F
          slow=1, fast=3

Step 3:  arr[F]=2 == arr[S]=2  →  duplicate, skip
         [1,  2,  2,  2,  3,  4,  4]
              S           F
          slow=1, fast=4

Step 4:  arr[F]=3 != arr[S]=2  →  new value! S++, copy
         [1,  2,  3,  2,  3,  4,  4]
                  S           F
          slow=2, fast=5

Step 5:  arr[F]=4 != arr[S]=3  →  new value! S++, copy
         [1,  2,  3,  4,  3,  4,  4]
                      S           F
          slow=3, fast=6

Step 6:  arr[F]=4 == arr[S]=4  →  duplicate, skip
         [1,  2,  3,  4,  3,  4,  4]
                      S               F (out of bounds)
          slow=3, fast=7 → DONE

Result:  first (slow+1) = 4 elements are unique: [1, 2, 3, 4, ...]
         New length = 4
```

#### Floyd's Cycle Detection (Tortoise and Hare)

A special case of fast/slow pointers for **linked list cycle detection**.

```
Idea:
  - Slow pointer moves 1 step at a time
  - Fast pointer moves 2 steps at a time
  - If there's a cycle, fast will eventually "lap" slow
  - If no cycle, fast hits NULL

         ┌──→ [3] → [4] → [5] ──┐
         │                       │
  [1] → [2]                     ↓
                                [6]
         ↑                       │
         └──── [8] ← [7] ←──────┘

  Slow: 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 3 → ...
  Fast: 1 → 3 → 5 → 7 → 3 → 5 → 7 → 3 → ...
                                          ↑
                              They meet at node 3 (or 5 or 7)

  Meeting proves a cycle exists!
```

**Finding the cycle start**: After they meet, move one pointer back to head. Advance both at speed 1. They meet at the **cycle entry point**. (Mathematical proof relies on the distance relationship.)

#### Other Classic Problems

| Problem | Key Idea |
|---|---|
| **Remove element in-place** | Slow = write pos, fast skips target value |
| **Move zeroes** | Slow = next non-zero position, fast scans |
| **Find middle of linked list** | When fast hits end, slow is at middle |
| **Linked list cycle entry** | Floyd's algorithm extended |
| **Happy number** | Treat as implicit linked list, detect cycle |

#### Template Code — Fast/Slow

```cpp
// Remove duplicates from sorted array in-place
// Returns new length of the deduplicated portion
int removeDuplicates(vector<int>& arr) {
    if (arr.empty()) return 0;

    int slow = 0;   // last unique element position

    for (int fast = 1; fast < arr.size(); fast++) {
        if (arr[fast] != arr[slow]) {
            slow++;
            arr[slow] = arr[fast];  // write new unique value
        }
        // if equal, fast just advances (skip duplicate)
    }
    return slow + 1;  // length = last index + 1
}
```

```cpp
// Floyd's cycle detection in a linked list
bool hasCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;

    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;          // 1 step
        fast = fast->next->next;    // 2 steps

        if (slow == fast) return true;  // they met → cycle
    }
    return false;  // fast hit end → no cycle
}
```

```
General pattern:

    int slow = 0;
    for (int fast = 0; fast < n; fast++) {
        if (meetsSomeCondition(arr[fast])) {
            arr[slow] = arr[fast];
            slow++;
        }
    }
    // slow = count of elements that met condition
```

---

### Pattern 3: Two Arrays Pointers

Each pointer belongs to a **different array** (or different list). Both advance independently based on comparisons.

**When to use**: Merging sorted sequences, finding intersections, comparing sorted data.

#### ASCII Walkthrough — Merge Two Sorted Arrays

**Problem**: Merge `[1, 3, 5]` and `[2, 4, 6]` into a single sorted array.

```
Arrays:   A = [1, 3, 5]     B = [2, 4, 6]     result = []
               i                  j

Step 1:   A[i]=1 < B[j]=2  →  pick 1, advance i
          A = [1, 3, 5]     B = [2, 4, 6]     result = [1]
                  i              j

Step 2:   A[i]=3 > B[j]=2  →  pick 2, advance j
          A = [1, 3, 5]     B = [2, 4, 6]     result = [1, 2]
                  i                  j

Step 3:   A[i]=3 < B[j]=4  →  pick 3, advance i
          A = [1, 3, 5]     B = [2, 4, 6]     result = [1, 2, 3]
                     i           j

Step 4:   A[i]=5 > B[j]=4  →  pick 4, advance j
          A = [1, 3, 5]     B = [2, 4, 6]     result = [1, 2, 3, 4]
                     i               j

Step 5:   A[i]=5 < B[j]=6  →  pick 5, advance i
          A = [1, 3, 5]     B = [2, 4, 6]     result = [1, 2, 3, 4, 5]
                        i        j
                        ↑
                    i past end of A → copy remaining B

Step 6:   Copy remaining B elements
          result = [1, 2, 3, 4, 5, 6]     ✓ DONE
```

#### Other Classic Problems

| Problem | Key Idea |
|---|---|
| **Intersection of sorted arrays** | Advance the pointer at the smaller value; collect when equal |
| **Merge step of Merge Sort** | Exact same pattern as above |
| **Compare sorted lists** | Check equality element by element |
| **Shortest word distance** | Track positions in two sorted index lists |

#### Template Code — Two Arrays

```cpp
// Merge two sorted arrays into one sorted result
vector<int> mergeSorted(vector<int>& A, vector<int>& B) {
    vector<int> result;
    int i = 0, j = 0;

    while (i < A.size() && j < B.size()) {
        if (A[i] <= B[j]) {
            result.push_back(A[i]);
            i++;
        } else {
            result.push_back(B[j]);
            j++;
        }
    }

    // copy remaining elements from whichever array isn't exhausted
    while (i < A.size()) result.push_back(A[i++]);
    while (j < B.size()) result.push_back(B[j++]);

    return result;
}
```

```cpp
// Intersection of two sorted arrays (unique elements)
vector<int> intersection(vector<int>& A, vector<int>& B) {
    vector<int> result;
    int i = 0, j = 0;

    while (i < A.size() && j < B.size()) {
        if (A[i] == B[j]) {
            result.push_back(A[i]);
            i++;
            j++;
        } else if (A[i] < B[j]) {
            i++;    // A's element is smaller, advance A
        } else {
            j++;    // B's element is smaller, advance B
        }
    }
    return result;
}
```

```
General pattern:

    int i = 0, j = 0;
    while (i < A.size() && j < B.size()) {
        // compare A[i] and B[j]
        // advance the appropriate pointer
    }
    // handle remaining elements in whichever array isn't done
```

---

## 🔹 When to Use Two Pointers — Pattern Recognition Checklist

Use this mental checklist when you see a problem. If 1-2 of these apply, two pointers is likely the right approach:

| Signal | Pattern |
|---|---|
| Input is **sorted** (or can be sorted without breaking the problem) | Opposite-direction or two-arrays |
| Need to find a **pair/triplet** with a specific sum or property | Opposite-direction |
| Need to do something **in-place** (O(1) extra space) | Fast/slow |
| Need to **remove/filter elements** without extra array | Fast/slow |
| Working with a **linked list** (cycle, middle, nth from end) | Fast/slow |
| Need to **merge or compare** two sorted sequences | Two-arrays |
| Problem says "O(n) time" and brute force is O(n^2) pair checking | Strong hint for two pointers |
| **Palindrome** or **symmetry** check | Opposite-direction |
| **Partitioning** an array (Dutch National Flag, etc.) | Multiple pointers variant |

---

## 🔹 Complexity Analysis

| Aspect | Typical | Why |
|---|---|---|
| **Time** | O(n) | Each pointer traverses the array at most once. Total work = O(n) + O(n) = O(n) |
| **Space** | O(1) | Only two index variables needed (no extra data structures) |
| **vs. Brute Force** | O(n) vs O(n^2) | Brute force checks all pairs; two pointers eliminates invalid pairs structurally |

**Exception**: If the problem requires sorting first and the input is unsorted, total time becomes O(n log n) due to the sort, with the two-pointer scan itself still O(n).

```
Brute Force vs Two Pointers — visual comparison:

  Brute Force (nested loops):       Two Pointers:
  ┌──────────────────────┐          ┌──────────────────────┐
  │ x x x x x x x x x x │  O(n^2) │ L → → → → →   ← ← R │  O(n)
  │ x x x x x x x x x x │         │                       │
  │ x x x x x x x x x x │         │  Each pointer moves   │
  │ x x x x x x x x x x │         │  at most n times      │
  │ x x x x x x x x x x │         │                       │
  │ x x x x x x x x x x │         │  Total: ~2n steps     │
  │ x x x x x x x x x x │         │                       │
  └──────────────────────┘          └──────────────────────┘
   Every pair checked                Smart elimination
```

---

## 🔹 Common Pitfalls

### 1. Forgetting the Sorted Prerequisite
Opposite-direction two pointers **only work on sorted input**. If the array is unsorted, you must sort it first — but watch out: sorting may lose original indices. If you need original indices, use a hash map instead.

```
WRONG: Two Sum on unsorted array with two pointers
  [3, 1, 4, 6, 2]  →  L=3, R=2, sum=5 ... pointers don't know
                        which direction reduces/increases sum!

RIGHT: Sort first, or use a hash map if indices matter.
```

### 2. Off-by-One Errors on Boundaries
The most common bug. Double-check:
- `while (left < right)` vs `while (left <= right)` — usually `<` for pairs, `<=` for single-element searches
- Whether to process `arr[left]` before or after incrementing
- Whether the result length is `slow` or `slow + 1`

### 3. Infinite Loops — Pointers Must Always Advance
Every branch of your while loop **must move at least one pointer**. If neither pointer moves, you loop forever.

```cpp
// BUG: missing pointer advancement
while (left < right) {
    int sum = arr[left] + arr[right];
    if (sum == target) {
        return {left, right};
    }
    // OOPS: if sum != target and we forget to move a pointer → infinite loop
}
```

### 4. Not Handling Duplicates
When the problem says "unique results" (like 3Sum), you must skip duplicate values after finding a match:

```cpp
// Skip duplicates after finding a valid pair
if (sum == target) {
    result.push_back({arr[left], arr[right]});
    while (left < right && arr[left] == arr[left + 1]) left++;
    while (left < right && arr[right] == arr[right - 1]) right--;
    left++;
    right--;
}
```

### 5. Modifying the Array When You Shouldn't
Fast/slow pointer techniques modify the array in-place. Make sure the problem actually wants in-place modification. If you need the original array later, make a copy first.

---

## 🔹 Quick Reference — Choosing the Right Pattern

```
                    ┌─────────────────────────┐
                    │   Is the input sorted?   │
                    └────────┬────────────────┘
                             │
                  ┌──────────┴──────────┐
                  │ YES                 │ NO
                  ▼                     ▼
         ┌────────────────┐    ┌──────────────────┐
         │ One array or   │    │ Can you sort it   │
         │ two arrays?    │    │ without losing     │
         └───┬────────┬───┘    │ needed info?       │
             │        │        └───┬────────────┬───┘
          ONE       TWO         YES            NO
             │        │           │              │
             ▼        ▼           ▼              ▼
       ┌──────────┐ ┌────────┐ ┌──────┐   ┌──────────┐
       │Find pair?│ │Merge / │ │Sort  │   │Use hash  │
       │Palindrome│ │Intersect│ │first │   │map or    │
       │Container?│ │Compare │ │then  │   │other     │
       └────┬─────┘ └───┬────┘ │2-ptr │   │technique │
            │            │      └──┬───┘   └──────────┘
            ▼            ▼        ▼
       Opposite      Two-Arrays  Opposite
       Direction     Pointers    Direction
       Pointers

       ┌─────────────────────────────────────────┐
       │ Need in-place / linked list / cycle?     │
       │ → Fast/Slow pointers (any sort order)    │
       └─────────────────────────────────────────┘
```

---

## 🔹 Related Techniques

- [[Sliding Window Technique]] — A specialized form of same-direction two pointers where you maintain a window/range. When the window size varies, it's essentially fast/slow pointers tracking window boundaries.
- [[Binary Search]] — Also eliminates half the search space per step, but uses a single pointer (mid) rather than two. Sometimes combined with two pointers (e.g., binary search + two pointers for 3Sum).
- [[Merge Sort]] — The merge step is literally the two-arrays pointer pattern. Understanding two pointers deeply makes merge sort intuitive.
