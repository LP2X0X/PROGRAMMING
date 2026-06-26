---
tags:
  - algorithms
  - big-o
  - complexity-analysis
---

# Big O Notation

> **80/20 Concept** — Big O is the universal language for talking about algorithm efficiency. Understanding it deeply is the single most important prerequisite for all of DSA. Every algorithm choice, every data structure selection, and every interview answer depends on your ability to analyze and compare growth rates.

---

## 🔹 What Is Big O?

**Big O notation** describes how an algorithm's resource usage (time or space) **scales** as the input size `n` grows toward infinity. It gives you the **upper bound** on growth rate — the worst-case guarantee.

### Real-World Analogy

Imagine you're cooking for guests:

- **O(1):** Making a pot of rice. Whether it's for 1 guest or 100 guests, you use one rice cooker. The cooking time doesn't change with guest count (assuming the pot is big enough).
- **O(n):** Handing out plates. Each guest needs one plate — double the guests, double the work.
- **O(n^2):** Every guest shakes hands with every other guest. 10 guests = ~100 handshakes. 100 guests = ~10,000 handshakes. It explodes.

Big O captures this intuition mathematically: ==how does work scale with input size?==

### The Formal Definition

If a function `f(n)` is **O(g(n))**, it means:

> There exist constants **c > 0** and **n0 >= 0** such that for all **n >= n0**:
> **f(n) <= c * g(n)**

**In plain English:** Beyond some point, `f(n)` never grows faster than a constant multiple of `g(n)`. We say `g(n)` is an **upper bound** on `f(n)`.

```
f(n)
  ^
  |         c * g(n)
  |        /
  |      /    <-- f(n) stays below this line
  |    /         for all n >= n0
  |  / . . f(n)
  | /.·
  |/ 
  +---|-----------> n
     n0
```

### Why We Drop Constants and Lower-Order Terms

Consider a function that counts exact operations:

```
f(n) = 3n^2 + 5n + 2
```

As `n` grows:

| n | 3n^2 | 5n | 2 | Total | % from n^2 |
|---|---|---|---|---|---|
| 10 | 300 | 50 | 2 | 352 | 85% |
| 100 | 30,000 | 500 | 2 | 30,502 | 98% |
| 1,000 | 3,000,000 | 5,000 | 2 | 3,005,002 | 99.8% |
| 10,000 | 300,000,000 | 50,000 | 2 | 300,050,002 | 99.98% |

The `n^2` term **completely dominates** as n grows. The `5n + 2` becomes insignificant noise. The constant `3` doesn't affect the *shape* of the growth — it just shifts the curve vertically.

**Therefore:** `3n^2 + 5n + 2` simplifies to **O(n^2)**.

**The rules:**

1. **Drop constants:** `O(3n)` = `O(n)`, because constants don't change the growth *shape*
2. **Drop lower-order terms:** `O(n^2 + n)` = `O(n^2)`, because the biggest term dominates
3. **Keep different variables:** `O(n + m)` stays as `O(n + m)` — you can't drop `m` because it might be larger than `n`

> [!warning] Common Misconception
> "Dropping constants means constants don't matter." **Wrong.** Constants matter enormously in practice. An O(n) algorithm with a constant factor of 1000 is slower than an O(n^2) algorithm for n < 1000. Big O tells you how things scale *asymptotically* — for large n. For small n, measure and benchmark. See [[Constraint Analysis]] for how to use input size to pick the right approach.

---

## 🔹 Common Complexities

### Master Table

| Notation | Name | Example Algorithm | Growth at n=1000 |
|---|---|---|---|
| **O(1)** | Constant | Array access, hash lookup | 1 |
| **O(log n)** | Logarithmic | [[Binary Search]] | ~10 |
| **O(n)** | Linear | Single loop, linear scan | 1,000 |
| **O(n log n)** | Linearithmic | [[Merge Sort]], [[Quick Sort]] avg | ~10,000 |
| **O(n^2)** | Quadratic | Nested loops, bubble sort | 1,000,000 |
| **O(n^3)** | Cubic | 3 nested loops, naive matrix mult | 1,000,000,000 |
| **O(2^n)** | Exponential | Recursive fibonacci, subsets | ~10^301 |
| **O(n!)** | Factorial | Permutations, brute force TSP | ~10^2567 |

---

### O(1) — Constant Time

The algorithm does the **same amount of work** regardless of input size.

```cpp
int getFirst(int arr[], int n) {
    return arr[0];    // One operation, always
}

// Hash table lookup (average case)
int value = hashMap[key];    // O(1) average

// Stack push/pop
stack.push(42);    // O(1)
stack.pop();       // O(1)
```

**Why it's O(1):** The number of operations doesn't depend on `n` at all. Whether the array has 10 elements or 10 million, accessing index 0 is one memory lookup.

**Common O(1) operations:**
- Array access by index: `arr[i]`
- Hash table get/set (average case)
- Stack push/pop
- Queue enqueue/dequeue (with proper implementation)
- Linked list insert/delete at head
- Arithmetic operations

---

### O(log n) — Logarithmic Time

The algorithm **halves the problem** with each step. This is the "magic" complexity — it handles massive inputs with surprisingly few steps.

```cpp
int binarySearch(int arr[], int n, int target) {
    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

**Why halving = logarithmic:**

Each step eliminates half the remaining elements. Starting with `n` elements:

```
Step 0: n elements
Step 1: n/2 elements
Step 2: n/4 elements
Step 3: n/8 elements
...
Step k: n / 2^k elements

We stop when n / 2^k = 1
=> 2^k = n
=> k = log2(n)
```

**How many steps for real inputs:**

| Input size n | log2(n) steps | That's this many comparisons |
|---|---|---|
| 1,000 | ~10 | Ten! |
| 1,000,000 | ~20 | Twenty! |
| 1,000,000,000 | ~30 | Thirty! |
| 4,000,000,000 | ~32 | Every int in existence |

```
Search space shrinking (n = 16):

Step 0: [■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■]  16 elements
Step 1: [■ ■ ■ ■ ■ ■ ■ ■ · · · · · · · ·]   8 elements
Step 2: [■ ■ ■ ■ · · · · · · · · · · · ·]   4 elements
Step 3: [■ ■ · · · · · · · · · · · · · ·]   2 elements
Step 4: [■ · · · · · · · · · · · · · · ·]   1 element  --> FOUND
```

> [!tip] The log n Intuition
> Whenever you see an algorithm that **divides the problem in half** (or thirds, or any fraction) at each step, think **O(log n)**. The base of the logarithm doesn't matter in Big O because `log_2(n)` and `log_10(n)` differ by only a constant factor.

**Common O(log n) operations:**
- [[Binary Search]] on a sorted array
- BST lookup/insert/delete (balanced tree)
- Finding an element in a balanced [[Binary Search Tree]]

---

### O(n) — Linear Time

The algorithm does work **proportional** to the input size. Double the input, double the work.

```cpp
int findMax(int arr[], int n) {
    int maxVal = arr[0];
    for (int i = 1; i < n; i++) {    // visits every element once
        if (arr[i] > maxVal)
            maxVal = arr[i];
    }
    return maxVal;
}
```

**Why it's O(n):** The loop runs exactly `n - 1` times. Each iteration does O(1) work. Total: `(n - 1) * O(1)` = O(n).

**Common O(n) operations:**
- Linear search through an unsorted array
- Traversing a linked list
- Counting elements
- Finding min/max
- Single-pass hash map construction

---

### O(n log n) — Linearithmic Time

The algorithm does **n work at each of log n levels**. This is the sweet spot for efficient sorting.

```
Why Merge Sort is O(n log n):

Level 0:  [  n elements to merge  ]             n work
Level 1:  [ n/2 ] [ n/2 ]                       n work
Level 2:  [n/4][n/4] [n/4][n/4]                  n work
Level 3:  [n/8]...[n/8]  (8 groups)              n work
...
Level log n: [1][1][1]...[1]  (n groups)         n work
             ─────────────────────────────────
                                        Total: n * log n
```

**Intuitive explanation:** [[Merge Sort]] splits the array in half recursively (log n levels deep), and at each level it does a total of n comparisons during the merge step. Hence `n * log n` total work.

**Common O(n log n) operations:**
- [[Merge Sort]]
- [[Quick Sort]] (average case)
- Heap Sort
- Building a balanced BST from an unsorted array

---

### O(n^2) — Quadratic Time

The algorithm compares **every element to every other element**. Extremely common with nested loops.

```cpp
// Find all pairs
void printAllPairs(int arr[], int n) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            printf("(%d, %d) ", arr[i], arr[j]);
        }
    }
}
```

**Why it's O(n^2):** The outer loop runs `n` times. For each outer iteration, the inner loop runs `n` times. Total iterations: `n * n = n^2`.

```
All pairs for n = 5 (each dot = one operation):

     j=0  j=1  j=2  j=3  j=4
i=0   .    .    .    .    .      5 operations
i=1   .    .    .    .    .      5 operations
i=2   .    .    .    .    .      5 operations
i=3   .    .    .    .    .      5 operations
i=4   .    .    .    .    .      5 operations
                                 ──────────
                                 25 = 5^2 total
```

**Common O(n^2) operations:**
- Bubble Sort, Selection Sort, Insertion Sort (worst case)
- Comparing all pairs in an array
- Naive string matching
- Matrix operations (simple loops)

> [!warning] Common Misconception
> "Two nested loops always means O(n^2)." **Not necessarily.** If the inner loop runs a *constant* number of times, it's still O(n). And if the inner loop uses a different variable, it's O(n * m), not O(n^2). Always analyze the actual loop bounds.

---

### O(2^n) — Exponential Time

The algorithm's work **doubles** with each additional input element. These blow up fast.

```cpp
// Naive recursive Fibonacci
int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}
```

**Recursion tree for fib(5):**

```
                         fib(5)
                        /      \
                   fib(4)       fib(3)
                  /     \       /     \
             fib(3)  fib(2)  fib(2)  fib(1)
             /   \    /  \    /  \
         fib(2) fib(1) fib(1) fib(0) fib(1) fib(0)
         /   \
     fib(1) fib(0)
```

Each level roughly doubles the number of calls. For `fib(n)`, there are approximately `2^n` total calls.

**How fast it grows:**

| n | 2^n |
|---|---|
| 10 | 1,024 |
| 20 | 1,048,576 (~1 million) |
| 30 | 1,073,741,824 (~1 billion) |
| 40 | ~1 trillion |
| 50 | ~1 quadrillion |

This is why naive Fibonacci is practically useless for n > ~40. Use [[Memoization vs Tabulation|dynamic programming]] to reduce it to O(n).

**Common O(2^n) operations:**
- Generating all subsets of a set
- Naive recursive Fibonacci
- Brute-force solutions to many NP problems

---

### O(n!) — Factorial Time

The algorithm considers **all possible orderings**. This is typically the brute-force approach to permutation problems.

```cpp
// Generate all permutations
void permute(int arr[], int start, int end) {
    if (start == end) {
        printArray(arr, end + 1);
        return;
    }
    for (int i = start; i <= end; i++) {
        swap(arr[start], arr[i]);
        permute(arr, start + 1, end);
        swap(arr[start], arr[i]);  // backtrack
    }
}
```

**How absurdly fast it grows:**

| n | n! |
|---|---|
| 5 | 120 |
| 10 | 3,628,800 |
| 13 | 6,227,020,800 (~6 billion) |
| 15 | 1,307,674,368,000 (~1.3 trillion) |
| 20 | ~2.4 * 10^18 |

At n = 20, even a computer doing a billion operations per second would take **77 years**.

---

## 🔹 Visual Growth Comparison

This table makes the differences viscerally clear. Assume each operation takes 1 microsecond (10^-6 seconds):

| n | O(1) | O(log n) | O(n) | O(n log n) | O(n^2) | O(2^n) | O(n!) |
|---|---|---|---|---|---|---|---|
| 10 | 1 us | 3 us | 10 us | 33 us | 100 us | 1 ms | 3.6 s |
| 100 | 1 us | 7 us | 100 us | 664 us | 10 ms | 10^14 yrs | heat death++ |
| 1,000 | 1 us | 10 us | 1 ms | 10 ms | 1 s | heat death++ | heat death++ |
| 10,000 | 1 us | 13 us | 10 ms | 133 ms | 100 s | heat death++ | heat death++ |
| 100,000 | 1 us | 17 us | 100 ms | 1.7 s | 2.8 hrs | heat death++ | heat death++ |
| 1,000,000 | 1 us | 20 us | 1 s | 20 s | 11.6 days | heat death++ | heat death++ |

```
Growth rate visualization:

operations
    ^
    |                                            * O(n!)
    |                                          *
    |                                       *    * O(2^n)
    |                                     *   *
    |                                   *  *
    |                                 **
    |                              * *
    |                           **       ..... O(n^2)
    |                        * *     ....
    |                      **    ....
    |                   * *  ....
    |               ****....            ___--- O(n log n)
    |          ** ..            ___---
    |     *  ..        ___---
    |  * ..    ___---             ____________ O(n)
    | ..___---              _____
    |---          _____-----     _____________ O(log n)
    |_____-----
    |____________________________________________ O(1)
    +----------------------------------------------> n
```

> [!danger] The Wall
> Once you hit exponential or factorial complexity, no amount of hardware can save you. A 2x faster computer only adds 1 to the max n for O(2^n). The only solution is a **better algorithm** — often dynamic programming or greedy approaches.

---

## 🔹 How to Analyze Code — Step by Step

This is the most practical skill. Given a piece of code, determine its Big O.

### Rule 1: Single Loop = O(n)

```cpp
for (int i = 0; i < n; i++) {
    // O(1) work per iteration
}
```

**Analysis:** Loop runs `n` times. Each iteration is O(1). Total: **O(n)**.

---

### Rule 2: Nested Loops = Multiply

```cpp
for (int i = 0; i < n; i++) {         // O(n)
    for (int j = 0; j < n; j++) {     //   * O(n)
        // O(1) work
    }
}
```

**Analysis:** Outer loop runs `n` times. For each, inner loop runs `n` times. Total: **O(n * n) = O(n^2)**.

---

### Rule 3: Loop with Halving = O(log n)

```cpp
int i = n;
while (i > 0) {
    // O(1) work
    i = i / 2;
}
```

**Analysis:** `i` starts at `n` and is halved each step. Steps to reach 0: `log2(n)`. Total: **O(log n)**.

Also applies to doubling:

```cpp
for (int i = 1; i < n; i = i * 2) {
    // O(1) work
}
```

Same thing — `i` doubles each step, reaching `n` in `log2(n)` steps: **O(log n)**.

---

### Rule 4: Different Variables = Keep Both

```cpp
for (int i = 0; i < n; i++) {         // O(n)
    for (int j = 0; j < m; j++) {     //   * O(m)
        // O(1) work
    }
}
```

**Analysis:** Outer runs `n` times, inner runs `m` times. Total: **O(n * m)**. You CANNOT simplify this to O(n^2) unless you know n == m.

---

### Rule 5: Sequential Blocks = Take the Max

```cpp
// Block A
for (int i = 0; i < n; i++) {         // O(n)
    // O(1) work
}

// Block B
for (int i = 0; i < n; i++) {         // O(n)
    for (int j = 0; j < n; j++) {     //   * O(n)
        // O(1) work
    }
}
```

**Analysis:** Block A is O(n). Block B is O(n^2). Total: O(n) + O(n^2) = **O(n^2)** (the larger term dominates).

---

### Rule 6: Recursive Functions — Write the Recurrence

```cpp
void solve(int n) {
    if (n <= 1) return;       // base case
    solve(n / 2);             // one recursive call, half the input
    solve(n / 2);             // another recursive call, half the input
    doLinearWork(n);          // O(n) work at this level
}
```

**Step 1 — Write the recurrence:**
```
T(n) = 2 * T(n/2) + O(n)
       ──────────   ────
       2 calls       work done at
       each half     this level
       the size
```

**Step 2 — Solve it (or use the Master Theorem):**
This is the [[Merge Sort]] recurrence. Answer: **T(n) = O(n log n)**.

See the Master Theorem section below for the systematic approach.

---

### Rule 7: Amortized Analysis — Average Over Many Operations

Some operations are expensive occasionally but cheap most of the time. Example: dynamic array append.

```cpp
vector<int> vec;
for (int i = 0; i < n; i++) {
    vec.push_back(i);    // Is this O(1) or O(n)?
}
```

Most `push_back` calls are O(1) — just write to the next slot. But when the array is full, it must allocate a new array of double the size and copy everything: O(n). However, this expensive operation happens so rarely that the **average cost per operation** is O(1).

**Total cost of n push_back operations:** O(n)
**Amortized cost per operation:** O(n) / n = **O(1) amortized**

(See the full amortized analysis section below.)

---

### Worked Example: Putting It All Together

```cpp
void mystery(int arr[], int n) {
    // Block 1: O(n)
    for (int i = 0; i < n; i++) {
        arr[i] = arr[i] * 2;
    }

    // Block 2: O(n^2)
    for (int i = 0; i < n; i++) {
        for (int j = i; j < n; j++) {    // Note: j starts at i
            if (arr[i] > arr[j])
                swap(arr[i], arr[j]);
        }
    }

    // Block 3: O(log n)
    int x = n;
    while (x > 1) {
        printf("%d ", x);
        x = x / 2;
    }
}
```

**Block 1:** Simple loop, O(n).

**Block 2:** Nested loop, but the inner loop starts at `i`, not 0.
- When i=0: inner runs n times
- When i=1: inner runs n-1 times
- When i=2: inner runs n-2 times
- ...
- When i=n-1: inner runs 1 time
- Total: n + (n-1) + (n-2) + ... + 1 = **n(n+1)/2 = O(n^2)**

**Block 3:** Halving loop, O(log n).

**Total:** O(n) + O(n^2) + O(log n) = **O(n^2)** (take the dominant term).

---

## 🔹 Space Complexity

Space complexity measures **how much extra memory** the algorithm uses as a function of input size.

### What Counts as Space

1. **Auxiliary space** — extra memory the algorithm allocates (arrays, hash maps, etc.)
2. **Stack space** — memory consumed by recursive calls (each call adds a frame to the call stack)

> [!info] Definition
> **Auxiliary space complexity** = only the extra space (not counting the input).
> **Total space complexity** = auxiliary space + input space.
> In interviews, "space complexity" usually means **auxiliary space** unless stated otherwise.

### Common Space Complexity Examples

| Scenario | Space | Why |
|---|---|---|
| In-place sorting ([[Quick Sort]]) | O(log n) | Only recursion stack (log n deep) |
| [[Merge Sort]] | O(n) | Temporary array for merging |
| Hash map of all elements | O(n) | One entry per element |
| 2D DP table (n x m) | O(n * m) | Full grid stored |
| Recursive DFS on a tree | O(h) | Stack depth = tree height |
| Recursive DFS on a graph | O(V) | Visited set + recursion stack |
| Iterative algorithm with 3 pointers | O(1) | Fixed number of variables |

### Recursion Stack Space

Every recursive call adds a **stack frame**. The maximum recursion depth determines stack space:

```cpp
// O(n) stack space — recurses n times
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);    // n frames on the stack
}

// O(log n) stack space — recurses log n times
int binarySearch(int arr[], int left, int right, int target) {
    if (left > right) return -1;
    int mid = left + (right - left) / 2;
    if (arr[mid] == target) return mid;
    if (arr[mid] < target)
        return binarySearch(arr, mid + 1, right, target);
    return binarySearch(arr, left, mid - 1, target);
}
```

```
Factorial(5) call stack:           Binary Search call stack (max):

|  factorial(1)  |  <-- top       |  bSearch(mid+1, right)  |
|  factorial(2)  |                |  bSearch(left, right)   |
|  factorial(3)  |                |  bSearch(0, n-1)        |
|  factorial(4)  |                └──────────────────────────
|  factorial(5)  |                    3 frames for n = 8
└────────────────┘                    (log2(8) = 3)
    5 frames for n = 5
```

> [!tip] Tail Recursion
> Some recursive functions can be rewritten as loops (tail-call optimization) to eliminate stack space. C++ compilers sometimes do this automatically with optimization flags, but don't count on it — be explicit if you need O(1) space.

---

## 🔹 Best / Worst / Average Case

Big O can describe different scenarios for the same algorithm.

### Linear Search Example

```cpp
int linearSearch(int arr[], int n, int target) {
    for (int i = 0; i < n; i++) {
        if (arr[i] == target)
            return i;
    }
    return -1;
}
```

| Case | When | Complexity | Explanation |
|---|---|---|---|
| **Best** | Target is the first element | O(1) | Loop runs once |
| **Worst** | Target is last or not present | O(n) | Loop runs n times |
| **Average** | Target is at a random position | O(n) | Expected position: n/2, which is still O(n) |

### Which Case Does Big O Refer To?

By convention, **Big O usually describes the worst case** unless otherwise stated. This gives you a **guarantee** — the algorithm will NEVER be slower than this.

### The Other Notations

| Notation | Meaning | Analogy |
|---|---|---|
| **O(g(n))** — Big O | Upper bound (at most) | "This algorithm takes **at most** O(n^2) time" |
| **Omega(g(n))** — Big Omega | Lower bound (at least) | "This algorithm takes **at least** Omega(n) time" |
| **Theta(g(n))** — Big Theta | Tight bound (exactly) | "This algorithm takes **exactly** Theta(n log n) time" |

Theta means the algorithm is BOTH O(g(n)) AND Omega(g(n)) — the upper and lower bounds match.

**Example:** [[Merge Sort]] is **Theta(n log n)** because it always does exactly n log n work — best, worst, and average cases are identical. In contrast, [[Quick Sort]] is O(n^2) worst case but Theta(n log n) average case.

> [!tip] Practical Tip
> In interviews and casual conversation, people say "Big O" even when they mean Theta (tight bound). If someone says "Merge Sort is O(n log n)", they usually mean it's Theta(n log n). Technically O(n^2) is also true for Merge Sort (it's an upper bound, just not a tight one), but nobody says that because it's misleading.

---

## 🔹 Amortized Analysis

Some operations are usually cheap but occasionally expensive. **Amortized analysis** gives the average cost per operation over a sequence of operations.

### Dynamic Array (vector/ArrayList) Append

When a dynamic array runs out of space, it allocates a new array of **double the size** and copies everything over.

```
Appending n = 17 elements to a dynamic array (initial capacity 1):

Operation    Cost    Capacity After    Why
─────────────────────────────────────────────────────
push(1)      1       1 -> 2           fits, then resize (copy 1)
push(2)      1+1     2 -> 4           fits, then resize (copy 2)  -- wait
push(3)      1       4                fits
push(4)      1+2     4 -> 8           resize! copy 4 elements
push(5)      1       8                fits
push(6)      1       8                fits
push(7)      1       8                fits
push(8)      1+4     8 -> 16          resize! copy 8 elements
push(9-16)   1 each  16               fits (8 operations)
push(17)     1+8     16 -> 32         resize! copy 16 elements
```

Wait, let me redo this clearly. The key insight:

```
Capacity:    1    2    4    8    16   32

Resize at:   1    2    4    8    16
Copy cost:   1    2    4    8    16
             ──────────────────────
Total copy cost for n pushes:
1 + 2 + 4 + 8 + ... + n/2 + n  =  2n - 1  ≈  O(n)

Total cost = n (for the writes) + O(n) (for copies) = O(n)

Amortized cost per push = O(n) / n = O(1)
```

```
Cost per operation over time:

cost
  ^
  |
n |                                              #
  |
  |
  |
n/2                          #
  |
n/4              #
  |
  2      #
  1  # . # . . . # . . . . . . . # . . . . . . . . . . . . . . . #
  +--+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---> operation
     1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  16

     (#) = resize happened          (.) = simple O(1) append
     
     Total area of all bars ≈ 2n
     Average height = 2n / n = 2 = O(1) amortized
```

> [!tip] Banker's Method Intuition
> Think of each O(1) append as "depositing" an extra unit of work into a savings account. When a resize happens, you "withdraw" from the savings to pay for the expensive copy. Because resizes happen exponentially less often, the deposits always cover the withdrawals. Average cost: O(1).

---

## 🔹 Master Theorem (Simplified)

The Master Theorem gives you the Big O of **divide-and-conquer** recurrences of the form:

```
T(n) = a * T(n/b) + O(n^d)

Where:
  a = number of recursive calls
  b = factor by which input shrinks
  d = exponent of work done at each level
```

### The Three Cases

Compare `log_b(a)` with `d`:

| Case | Condition | Result | Intuition |
|---|---|---|---|
| **1** | `d < log_b(a)` | T(n) = O(n^(log_b(a))) | Recursion dominates — work multiplies faster than it shrinks |
| **2** | `d == log_b(a)` | T(n) = O(n^d * log n) | Balanced — equal work at each level, log n levels |
| **3** | `d > log_b(a)` | T(n) = O(n^d) | Top level dominates — most work is done at the root |

### Examples

**[[Merge Sort]]:** `T(n) = 2T(n/2) + O(n)`
- a = 2, b = 2, d = 1
- log_b(a) = log_2(2) = 1
- d == log_b(a) --> **Case 2:** T(n) = O(n^1 * log n) = **O(n log n)**

**[[Binary Search]]:** `T(n) = 1T(n/2) + O(1)`
- a = 1, b = 2, d = 0
- log_b(a) = log_2(1) = 0
- d == log_b(a) --> **Case 2:** T(n) = O(n^0 * log n) = **O(log n)**

**Strassen's Matrix Multiplication:** `T(n) = 7T(n/2) + O(n^2)`
- a = 7, b = 2, d = 2
- log_b(a) = log_2(7) = ~2.807
- d < log_b(a) --> **Case 1:** T(n) = O(n^2.807) = **O(n^2.807)**

**Naive tree traversal processing:** `T(n) = 2T(n/2) + O(n^2)`
- a = 2, b = 2, d = 2
- log_b(a) = log_2(2) = 1
- d > log_b(a) --> **Case 3:** T(n) = **O(n^2)**

> [!warning] When the Master Theorem Doesn't Apply
> The Master Theorem requires that all subproblems are the same size (n/b). It does NOT apply when:
> - Subproblems have different sizes (e.g., [[Quick Sort]] worst case: T(n) = T(n-1) + O(n))
> - The recurrence has non-polynomial extra work
> - There are multiple different-sized recursive calls
>
> For those cases, use the **recursion tree method** or **substitution method**.

---

## 🔹 Common Pitfalls

### 1. Hidden Complexity in Built-In Operations

```cpp
// TRAP: String concatenation in a loop
string result = "";
for (int i = 0; i < n; i++) {
    result += arr[i];    // Each += copies the ENTIRE string!
}
// Looks O(n), actually O(n^2) because each concatenation is O(current length)
// Lengths: 1 + 2 + 3 + ... + n = n(n+1)/2 = O(n^2)

// FIX: Use a string builder or reserve space first
```

```cpp
// TRAP: Searching in an unsorted container
for (int i = 0; i < n; i++) {
    if (vec.contains(arr[i])) {    // .contains() on a vector is O(n)!
        // ...
    }
}
// Looks O(n), actually O(n^2)

// FIX: Use a hash set for O(1) lookups
unordered_set<int> seen(vec.begin(), vec.end());    // O(n) to build
for (int i = 0; i < n; i++) {
    if (seen.count(arr[i])) {    // O(1) per lookup
        // ...
    }
}
// Now truly O(n)
```

### 2. Ignoring Recursion Stack Space

```cpp
int sum(int n) {
    if (n == 0) return 0;
    return n + sum(n - 1);    // O(n) stack frames!
}
// Time: O(n), Space: O(n) -- NOT O(1)!
```

### 3. Sorting Inside a Loop

```cpp
for (int i = 0; i < n; i++) {
    sort(arr, arr + n);    // O(n log n) inside an O(n) loop
    // ...
}
// Total: O(n^2 log n)  -- probably a bug
```

### 4. Confusing Average and Worst Case

```
Quick Sort:
  Average case: O(n log n)  <-- what people usually cite
  Worst case:   O(n^2)      <-- what actually happens on sorted input
                                with naive pivot selection

Hash table lookup:
  Average case: O(1)         <-- what people usually cite
  Worst case:   O(n)         <-- all keys hash to the same bucket
```

### 5. The "It's Just a Log" Trap

```cpp
// "log n is basically constant, right?"
// For n = 10^9:  log2(n) = 30
// For n = 10^18: log2(n) = 60

// 30-60x slower is NOT negligible when your inner loop runs billions of times
// O(n log n) vs O(n) can matter at scale
```

---

## 🔹 Practical Tips

### Use Constraint Analysis to Choose Algorithms

The input size `n` in a problem tells you which complexity you can afford. See [[Constraint Analysis]] for a complete table.

Quick reference:

| Max n | Required Complexity | Why |
|---|---|---|
| n <= 10 | O(n!) or O(2^n) | Brute force is fine |
| n <= 20 | O(2^n) | Bitmask DP |
| n <= 500 | O(n^3) | Triple nested loop |
| n <= 5,000 | O(n^2) | Double nested loop |
| n <= 100,000 | O(n log n) | Sort + sweep, divide and conquer |
| n <= 1,000,000 | O(n) or O(n log n) | Single pass, hash maps |
| n <= 10^9 | O(log n) or O(1) | Binary search, math formula |

### When Constants Actually Matter

Big O ignores constants, but in practice:

1. **Cache friendliness:** An O(n) array scan beats an O(n) linked list traversal by 10-100x because arrays are cache-friendly
2. **Small n:** For n < 50, an O(n^2) [[Simple Sorts|Insertion Sort]] often beats O(n log n) [[Merge Sort]] due to lower overhead — this is why production sorts (Timsort, Introsort) switch to Insertion Sort for small subarrays
3. **Constant factors:** An O(n log n) algorithm with a constant factor of 100 loses to an O(n^2) algorithm with a constant factor of 1 when n < ~1000

> [!tip] The Hierarchy of Optimization
> 1. **First:** Get the right Big O (choose the right algorithm)
> 2. **Second:** Reduce constant factors (cache locality, fewer allocations, simpler operations)
> 3. **Third:** Benchmark and profile actual performance
>
> Step 1 matters the most by far. Never micro-optimize an O(n^2) solution when an O(n log n) solution exists.

---

## 🔹 Summary

Big O is about understanding **how algorithms scale**. The key takeaways:

1. **Big O gives the upper bound** on growth rate — how the worst case scales with input size
2. **Drop constants and lower-order terms** because they become insignificant at large n
3. **Know the common complexities** — O(1), O(log n), O(n), O(n log n), O(n^2), O(2^n), O(n!)
4. **Analyze code systematically:** count loops, identify nesting vs sequence, write recurrences for recursion
5. **Don't forget space complexity** — especially recursion stack space
6. **Amortized analysis** handles operations that are "usually cheap, rarely expensive"
7. **Use constraint analysis** to choose the right complexity class for a problem

---

**See also:** [[Constraint Analysis]] | [[Sorting Comparison]] | [[Binary Search]] | [[Merge Sort]] | [[Quick Sort]] | [[Memoization vs Tabulation]]
