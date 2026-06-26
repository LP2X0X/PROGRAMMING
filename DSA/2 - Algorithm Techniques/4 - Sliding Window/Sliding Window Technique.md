---
tags:
  - algorithms
  - technique
  - sliding-window
---

## 🔹 Real-World Analogy

Imagine you are on a train, looking out through a fixed-width window. As the train moves, the window frame stays the same size, but the scenery you see changes — old scenery disappears from one edge, new scenery appears at the other. You never need to re-examine the entire landscape; you only note what enters and what leaves your view.

Another analogy: slide a magnifying glass across a line of text. At each position, you read the characters currently under the glass. Moving one position to the right means one character leaves the left edge and one new character appears on the right. You update your reading incrementally — you don't restart from scratch.

This is the core insight of sliding window: **incremental update instead of full recomputation**.

---

## 🔹 What Is Sliding Window?

Sliding window is a technique for processing **contiguous subarrays or substrings** efficiently. You maintain a "window" defined by two pointers (left and right) and slide it across the data structure.

The key optimization: instead of recalculating a property (sum, max, character count, etc.) from scratch for every possible subarray, you **update the result incrementally** by:
- **Adding** the new element entering the window (right side)
- **Removing** the element leaving the window (left side)

This turns an O(n*K) brute-force approach into an O(n) single-pass algorithm.

```
Brute force: recompute everything for each position
  [a  b  c  d  e  f  g]
   [-----]                 compute sum(a,b,c)
      [-----]              compute sum(b,c,d)  <-- redundant work!
         [-----]           compute sum(c,d,e)  <-- redundant work!

Sliding window: update incrementally
  [a  b  c  d  e  f  g]
   [-----]                 sum = a + b + c
      [-----]              sum = sum - a + d    <-- O(1) update
         [-----]           sum = sum - b + e    <-- O(1) update
```

---

## 🔹 Two Types of Sliding Window

There are two fundamental variants, each with its own pattern and template.

---

### 🔸 Type 1: Fixed-Size Window

The window size K is known in advance. You slide a window of exactly K elements across the array.

**Problem: Find the maximum sum of any subarray of size K = 3**

Array: `[2, 1, 5, 1, 3, 2]`

**Step-by-step ASCII walkthrough:**

```
Step 0: Build the initial window (first K=3 elements)
 
  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 1 ][ 5 ][ 1 ][ 3 ][ 2 ]
          [==========]
          L              R
 
  window_sum = 2 + 1 + 5 = 8
  max_sum    = 8


Step 1: Slide window right by 1 — remove arr[0], add arr[3]

  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 1 ][ 5 ][ 1 ][ 3 ][ 2 ]
          -         [==========]
          remove     L              R
                          add

  window_sum = 8 - arr[0] + arr[3]
             = 8 - 2 + 1 = 7
  max_sum    = max(8, 7) = 8


Step 2: Slide window right by 1 — remove arr[1], add arr[4]

  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 1 ][ 5 ][ 1 ][ 3 ][ 2 ]
                -         [==========]
                remove     L              R
                                add

  window_sum = 7 - arr[1] + arr[4]
             = 7 - 1 + 3 = 9
  max_sum    = max(8, 9) = 9          <-- new maximum!


Step 3: Slide window right by 1 — remove arr[2], add arr[5]

  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 1 ][ 5 ][ 1 ][ 3 ][ 2 ]
                          -         [==========]
                          remove     L              R
                                          add

  window_sum = 9 - arr[2] + arr[5]
             = 9 - 5 + 2 = 6
  max_sum    = max(9, 6) = 9


RESULT: Maximum subarray sum of size 3 = 9  (subarray [5, 1, 3])
```

**Why O(n) instead of O(n*K):**
- Brute force: for each of the (n - K + 1) positions, sum K elements → O(n * K)
- Sliding window: one pass through the array, each element is added once and removed once → O(n)

**C++ template — Fixed-Size Window:**

```cpp
// Fixed-size sliding window: find max sum of subarray of size K
int maxSumSubarray(vector<int>& arr, int K) {
    int n = arr.size();
    if (n < K) return -1;

    // Step 1: compute sum of first window
    int windowSum = 0;
    for (int i = 0; i < K; i++)
        windowSum += arr[i];

    int maxSum = windowSum;

    // Step 2: slide the window, one element at a time
    for (int i = K; i < n; i++) {
        windowSum += arr[i];       // add element entering window
        windowSum -= arr[i - K];   // remove element leaving window
        maxSum = max(maxSum, windowSum);
    }

    return maxSum;
}
```

---

### 🔸 Type 2: Variable-Size Window

The window size is **not fixed** — it grows and shrinks dynamically based on a condition. You expand the right pointer to explore, and contract the left pointer to optimize.

The general pattern:
1. **Expand**: move right pointer to include more elements until a condition is met
2. **Shrink**: move left pointer to find the optimal (smallest/largest) valid window
3. Repeat until the right pointer reaches the end

---

#### Problem 1: Smallest subarray with sum >= target

Array: `[2, 3, 1, 2, 4, 3]`, target = 7

**Step-by-step ASCII walkthrough:**

```
Initialize: left = 0, sum = 0, minLen = infinity

Step 1: Expand right to index 0
  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 3 ][ 1 ][ 2 ][ 4 ][ 3 ]
          [==]
           L,R
  sum = 2     (sum < 7, keep expanding)


Step 2: Expand right to index 1
  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 3 ][ 1 ][ 2 ][ 4 ][ 3 ]
          [=======]
           L    R
  sum = 5     (sum < 7, keep expanding)


Step 3: Expand right to index 2
  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 3 ][ 1 ][ 2 ][ 4 ][ 3 ]
          [============]
           L         R
  sum = 6     (sum < 7, keep expanding)


Step 4: Expand right to index 3
  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 3 ][ 1 ][ 2 ][ 4 ][ 3 ]
          [=================]
           L              R
  sum = 8     (sum >= 7! Record window size, try to shrink)

  minLen = min(inf, 4) = 4       window = [2,3,1,2]

  Shrink: remove arr[0]=2, left moves to 1
  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 3 ][ 1 ][ 2 ][ 4 ][ 3 ]
                [============]
                L         R
  sum = 6     (sum < 7, stop shrinking)


Step 5: Expand right to index 4
  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 3 ][ 1 ][ 2 ][ 4 ][ 3 ]
                [=================]
                L              R
  sum = 10    (sum >= 7! Record window size, try to shrink)

  minLen = min(4, 4) = 4        window = [3,1,2,4]

  Shrink: remove arr[1]=3, left moves to 2
  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 3 ][ 1 ][ 2 ][ 4 ][ 3 ]
                     [============]
                      L         R
  sum = 7     (sum >= 7! Record window size, keep shrinking)

  minLen = min(4, 3) = 3        window = [1,2,4]

  Shrink: remove arr[2]=1, left moves to 3
  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 3 ][ 1 ][ 2 ][ 4 ][ 3 ]
                          [=======]
                           L    R
  sum = 6     (sum < 7, stop shrinking)


Step 6: Expand right to index 5
  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 3 ][ 1 ][ 2 ][ 4 ][ 3 ]
                          [============]
                           L         R
  sum = 9     (sum >= 7! Record window size, try to shrink)

  minLen = min(3, 3) = 3        window = [2,4,3]

  Shrink: remove arr[3]=2, left moves to 4
  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 3 ][ 1 ][ 2 ][ 4 ][ 3 ]
                               [=======]
                                L    R
  sum = 7     (sum >= 7! Record window size, keep shrinking)

  minLen = min(3, 2) = 2        window = [4,3]     <-- new minimum!

  Shrink: remove arr[4]=4, left moves to 5
  Index:   0    1    2    3    4    5
  Array: [ 2 ][ 3 ][ 1 ][ 2 ][ 4 ][ 3 ]
                                    [==]
                                    L,R
  sum = 3     (sum < 7, stop shrinking)


Right pointer has reached the end. Done!
RESULT: Minimum subarray length with sum >= 7 is 2  (subarray [4, 3])
```

---

#### Problem 2: Longest substring without repeating characters

String: `"abcabcbb"`

**Step-by-step ASCII walkthrough:**

```
Initialize: left = 0, maxLen = 0, charSet = {}

Step 1: right = 0, char = 'a'
  String:  a   b   c   a   b   c   b   b
          [=]
           L,R
  'a' not in set → add it.   charSet = {a}
  maxLen = max(0, 1) = 1


Step 2: right = 1, char = 'b'
  String:  a   b   c   a   b   c   b   b
          [=====]
           L   R
  'b' not in set → add it.   charSet = {a, b}
  maxLen = max(1, 2) = 2


Step 3: right = 2, char = 'c'
  String:  a   b   c   a   b   c   b   b
          [=========]
           L       R
  'c' not in set → add it.   charSet = {a, b, c}
  maxLen = max(2, 3) = 3


Step 4: right = 3, char = 'a'
  String:  a   b   c   a   b   c   b   b
          [=============]
           L           R
  'a' IS in set! → Shrink from left until 'a' is removed:

    Remove arr[0]='a', left moves to 1
    String:  a   b   c   a   b   c   b   b
                [=========]
                L       R
    charSet = {b, c, a}
    (duplicate 'a' resolved — old 'a' removed, new 'a' now valid)

  maxLen = max(3, 3) = 3


Step 5: right = 4, char = 'b'
  String:  a   b   c   a   b   c   b   b
              [=============]
               L           R
  'b' IS in set! → Shrink from left until 'b' is removed:

    Remove arr[1]='b', left moves to 2
    String:  a   b   c   a   b   c   b   b
                    [=========]
                     L       R
    charSet = {c, a, b}
    (duplicate 'b' resolved)

  maxLen = max(3, 3) = 3


Step 6: right = 5, char = 'c'
  String:  a   b   c   a   b   c   b   b
                   [=============]
                    L           R
  'c' IS in set! → Shrink from left until 'c' is removed:

    Remove arr[2]='c', left moves to 3
    String:  a   b   c   a   b   c   b   b
                        [=========]
                         L       R
    charSet = {a, b, c}
    (duplicate 'c' resolved)

  maxLen = max(3, 3) = 3


Step 7: right = 6, char = 'b'
  String:  a   b   c   a   b   c   b   b
                       [=============]
                        L           R
  'b' IS in set! → Shrink from left until 'b' is removed:

    Remove arr[3]='a', left moves to 4   (charSet = {b, c})
    'b' still in set!
    Remove arr[4]='b', left moves to 5   (charSet = {c})
    Now add 'b':

    String:  a   b   c   a   b   c   b   b
                                [=====]
                                 L   R
    charSet = {c, b}

  maxLen = max(3, 2) = 3


Step 8: right = 7, char = 'b'
  String:  a   b   c   a   b   c   b   b
                                [=========]
                                 L       R
  'b' IS in set! → Shrink from left until 'b' is removed:

    Remove arr[5]='c', left moves to 6   (charSet = {b})
    'b' still in set!
    Remove arr[6]='b', left moves to 7   (charSet = {})
    Now add 'b':

    String:  a   b   c   a   b   c   b   b
                                        [=]
                                        L,R
    charSet = {b}

  maxLen = max(3, 1) = 3


RESULT: Longest substring without repeating characters = 3  ("abc")
```

**C++ template — Variable-Size Window:**

```cpp
// Variable-size: smallest subarray with sum >= target
int minSubarrayLen(int target, vector<int>& arr) {
    int n = arr.size();
    int left = 0;
    int sum = 0;
    int minLen = INT_MAX;

    for (int right = 0; right < n; right++) {
        sum += arr[right];               // expand: add right element

        while (sum >= target) {          // condition met? try to shrink
            minLen = min(minLen, right - left + 1);
            sum -= arr[left];            // shrink: remove left element
            left++;
        }
    }

    return (minLen == INT_MAX) ? 0 : minLen;
}
```

```cpp
// Variable-size: longest substring without repeating characters
int lengthOfLongestSubstring(string s) {
    unordered_set<char> charSet;
    int left = 0;
    int maxLen = 0;

    for (int right = 0; right < s.size(); right++) {
        while (charSet.count(s[right])) {   // duplicate found? shrink
            charSet.erase(s[left]);
            left++;
        }
        charSet.insert(s[right]);           // expand: add right char
        maxLen = max(maxLen, right - left + 1);
    }

    return maxLen;
}
```

---

## 🔹 How to Recognize Sliding Window Problems

Use this checklist when reading a problem statement:

| Signal | Example Phrasing |
|---|---|
| "Contiguous subarray" or "substring" | "Find a contiguous subarray that..." |
| Min/max length of subarray/substring | "Shortest substring containing..." |
| "Consecutive elements" | "Maximum of K consecutive elements" |
| Running aggregate over a range | "Sum/product/count in a window" |
| Fixed-size K subarray | "Every subarray of size K" |
| "At most K distinct" | "Longest substring with at most K distinct chars" |

**Decision flow:**

```
Is the problem about a contiguous subarray/substring?
├── NO  → Sliding window probably doesn't apply
└── YES
    ├── Is the window size given (fixed K)?
    │   └── YES → Fixed-size window
    └── NO, looking for min/max length?
        └── YES → Variable-size window
```

---

## 🔹 General Template Code

These templates cover the vast majority of sliding window problems. Adapt the condition and update logic to your specific problem.

```cpp
// ===== FIXED-SIZE WINDOW TEMPLATE =====
// Use when window size K is given
void fixedWindow(vector<int>& arr, int K) {
    int n = arr.size();

    // 1. Build initial window state from first K elements
    for (int i = 0; i < K; i++) {
        // ... add arr[i] to your window state
    }
    // ... record/check initial window result

    // 2. Slide the window
    for (int i = K; i < n; i++) {
        // ... add arr[i]       (new element entering)
        // ... remove arr[i-K]  (old element leaving)
        // ... update result
    }
}

// ===== VARIABLE-SIZE WINDOW TEMPLATE =====
// Use when looking for min/max length satisfying a condition
void variableWindow(vector<int>& arr) {
    int left = 0;
    int result = 0;  // or INT_MAX for min problems

    for (int right = 0; right < arr.size(); right++) {
        // 1. EXPAND: add arr[right] to window state

        // 2. SHRINK: while window is invalid (or valid, depending on problem)
        while (/* condition to shrink */) {
            // ... update result if needed
            // ... remove arr[left] from window state
            left++;
        }

        // 3. UPDATE: record result for current valid window
        // result = max(result, right - left + 1);  // for "longest"
        // result = min(result, right - left + 1);  // for "shortest" (inside while)
    }
}
```

---

## 🔹 Sliding Window vs Two Pointers

Sliding window **is a specific application** of the [[Two Pointers Technique]]. Here is how they relate:

| Aspect | Two Pointers (General) | Sliding Window |
|---|---|---|
| Pointer movement | Can move toward each other, away, same direction | Both pointers move in the **same direction** (left to right) |
| What's between pointers | May not matter | The elements between left and right form a **contiguous window** that you actively track |
| Data structure | Array, linked list, string | Array or string (must be indexable/contiguous) |
| Typical use | Pair sum, palindrome, merging | Subarray sum, substring problems |

```
Two Pointers (general):
  L -->            <-- R     (can move in opposite directions)
  L -->    R -->             (can move in same direction)

Sliding Window (specific case of two pointers):
  L -->    R -->             (always same direction)
  [========]                 (elements between L and R are
                              the "window" you maintain)
```

Think of it this way: every sliding window problem uses two pointers, but not every two-pointer problem is a sliding window. Sliding window requires that the contiguous range between the pointers is meaningful.

You can sometimes combine sliding window with [[Binary Search]] — for example, binary search on the answer (window size) and use a sliding window to check feasibility.

---

## 🔹 Complexity Analysis

**Time Complexity: O(n)**

The key insight: even though there is a nested `while` loop inside the `for` loop, the left pointer only moves forward. Across the entire algorithm:
- The right pointer moves from 0 to n-1: **n moves total**
- The left pointer moves from 0 to at most n-1: **at most n moves total**
- Each element is added to the window at most once and removed at most once

Total operations: at most 2n → **O(n)**

```
        right pointer moves: ──────────────────>  (n times)
        left pointer moves:  ──────────────────>  (at most n times)

        Each element: added once (when right passes it)
                      removed once (when left passes it)

        Total work per element: O(1) amortized
        Total: O(n)
```

**Space Complexity:**
- **O(1)** — if you only track a running sum/count
- **O(K)** — if you use a hash map/set to track elements in the window (e.g., character frequency map where K is the alphabet size or number of distinct elements)

---

## 🔹 Common Pitfalls

| Pitfall | What Goes Wrong | Fix |
|---|---|---|
| Forgetting to shrink | Variable-size window grows forever, never finds the optimal (minimum) answer | Always include the `while` shrink loop when condition is met |
| Off-by-one on window size | Computing `right - left` instead of `right - left + 1` | Window size with inclusive bounds is always `right - left + 1` |
| Wrong initial window (fixed-size) | Starting the slide loop at index 0 instead of K, double-counting elements | Build initial window with indices `[0, K-1]`, then slide starting at index K |
| Using sliding window when order matters | Problem needs non-contiguous elements or sorted subsequences | Sliding window only works for **contiguous** subarrays/substrings |
| Shrinking too much | Shrinking past the right pointer (`left > right`) | The while-loop condition should naturally prevent this, but verify edge cases |
| Fixed vs variable confusion | Using a fixed-size approach when the problem asks for minimum/maximum length | If the problem says "find the length", it is almost always variable-size |

---

## 🔹 Related Techniques

- [[Two Pointers Technique]] — the parent technique; sliding window is a specialization
- [[Binary Search]] — can be combined with sliding window (binary search on answer, validate with window)
