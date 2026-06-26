---
tags:
  - algorithms
  - technique
  - recursion
---
# Recursion

## 🔹 Real-World Analogy

**Russian nesting dolls (Matryoshka):** You open the biggest doll and find a smaller one inside. You open that one and find an even smaller one. You keep going until you reach the tiniest doll that doesn't open — that's the **base case**. Then you close them all back up in reverse order — that's the **unwinding** of the call stack.

**Dictionary lookup:** You look up the word "eloquent" and the definition says "fluent." You don't know "fluent" either, so you look that up too. Its definition says "smooth and effortless." Now you understand "fluent," so you go back and understand "eloquent." Each lookup is a recursive call; understanding the final word is the base case.

---

## 🔹 What is Recursion?

A function that **calls itself** to solve a problem by breaking it into smaller instances of the same problem.

Every recursive function has exactly two parts:

| Part | What it does | Analogy |
|------|-------------|---------|
| **Base case** | When to STOP recursing | The smallest doll that doesn't open |
| **Recursive case** | Call itself with a SMALLER input | Opening the next doll inside |

```
RECURSION = Base Case + Recursive Case
            (stop)      (shrink & repeat)
```

Without a base case, you get **infinite recursion** (stack overflow).
Without making the problem smaller, you also get infinite recursion.

---

## 🔹 The Call Stack

Every time a function calls itself, a new **stack frame** is pushed onto the call stack. Each frame holds its own local variables and waits for the deeper call to return.

### Example: `factorial(4)`

```
factorial(4) = 4 * factorial(3)
factorial(3) = 3 * factorial(2)
factorial(2) = 2 * factorial(1)
factorial(1) = 1                    ← base case
```

```
CALL STACK (growing downward as calls are made)
┌─────────────────────────┐
│ factorial(4)            │ ← waiting for factorial(3)
├─────────────────────────┤
│ factorial(3)            │ ← waiting for factorial(2)
├─────────────────────────┤
│ factorial(2)            │ ← waiting for factorial(1)
├─────────────────────────┤
│ factorial(1)            │ ← BASE CASE! returns 1
└─────────────────────────┘

UNWINDING (returns propagate upward):
factorial(1) = 1
factorial(2) = 2 * 1 = 2
factorial(3) = 3 * 2 = 6
factorial(4) = 4 * 6 = 24
```

```
Stack depth over time:

depth
  4 │          ┌───┐
  3 │       ┌──┤   ├──┐
  2 │    ┌──┤  │   │  ├──┐
  1 │ ┌──┤  │  │   │  │  ├──┐
  0 │─┘  │  │  │   │  │  │  └─
    └────┴──┴──┴───┴──┴──┴────→ time
     call call call BASE  unwind
      4    3    2   CASE   phase
```

**Key insight:** The stack grows during calls and shrinks during returns. Maximum stack depth = maximum recursion depth.

---

## 🔹 How to Think Recursively — THE KEY INSIGHT

> **"Trust the recursion."**

Assume the recursive call **already works correctly**. Your only job is:

1. **Handle the base case** — when is the problem trivially solvable?
2. **Do ONE step of work** — handle the current element/level
3. **Delegate the rest** — pass a smaller problem to the recursive call
4. **Combine** — merge your work with the recursive result

### Example: Reverse a linked list

Don't try to trace every call. Instead think:

```
reverse([1 → 2 → 3 → 4])

Step 1: Detach head (1) from rest (2 → 3 → 4)
Step 2: TRUST that reverse(2 → 3 → 4) = 4 → 3 → 2
Step 3: Attach 1 to the end:  4 → 3 → 2 → 1
Done!
```

You didn't need to think about how `reverse(2 → 3 → 4)` works internally. You **trusted** it and focused only on your current step.

### The Leap of Faith Method

```
1. What's the simplest input? (base case)
   → Empty list or single node → return it

2. If I had the answer for a SMALLER input, how would I build
   the answer for the CURRENT input?
   → If I have reverse(rest), just attach current node to the end

3. Write it!
```

---

## 🔹 Recursion vs Iteration

| Aspect | Recursion | Iteration |
|--------|-----------|-----------|
| Readability | More natural for trees/graphs | Better for simple loops |
| Space | O(n) stack space | O(1) typically |
| Speed | Function call overhead | Generally faster |
| Risk | Stack overflow on deep recursion | No stack overflow |
| Best for | Trees, divide & conquer, [[Backtracking]] | Arrays, simple repetition |
| Debugging | Harder to trace | Easier to step through |
| Convertible? | Every recursion can become iteration (with explicit stack) | Not all loops map naturally to recursion |

**Rule of thumb:**
- If the problem has a **tree/graph structure** → recursion is natural
- If it's a **linear scan** → iteration is simpler
- If recursion is deep (>10,000 levels) → convert to iteration or use tail recursion

---

## 🔹 Tail Recursion

A recursive call is **tail recursive** when the recursive call is the **very last operation** — nothing happens after it returns.

### Not tail recursive (normal):
```cpp
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);  // multiplication happens AFTER the call returns
}
```

### Tail recursive:
```cpp
int factorial(int n, int accumulator = 1) {
    if (n <= 1) return accumulator;
    return factorial(n - 1, n * accumulator);  // nothing left to do after this call
}
```

```
Trace of tail-recursive factorial(4):

factorial(4, 1)
  → factorial(3, 4)        // 4 * 1 = 4
    → factorial(2, 12)     // 3 * 4 = 12
      → factorial(1, 24)   // 2 * 12 = 24
        → return 24         // base case, accumulator has the answer
```

**Why it matters:** Some compilers (especially functional languages, and some C/C++ compilers with optimization flags) can transform tail recursion into a simple loop, using **O(1) stack space** instead of O(n). This is called **tail call optimization (TCO)**.

> Note: Not all languages guarantee TCO. C++ may do it with `-O2`, but don't rely on it for correctness.

---

## 🔹 Common Recursive Patterns

### 1. Factorial

```cpp
int factorial(int n) {
    if (n <= 1) return 1;           // base case
    return n * factorial(n - 1);    // recursive case
}
```

Time: O(n) | Space: O(n) call stack

---

### 2. Fibonacci (Naive)

```cpp
int fib(int n) {
    if (n <= 1) return n;               // base cases: fib(0)=0, fib(1)=1
    return fib(n - 1) + fib(n - 2);    // two recursive calls!
}
```

**Why this is O(2^n) — the recursion tree:**

```
                        fib(5)
                      /        \
                 fib(4)         fib(3)
                /     \         /     \
           fib(3)   fib(2)   fib(2)  fib(1)
           /   \     /   \    /   \
       fib(2) fib(1) fib(1) fib(0) fib(1) fib(0)
       /   \
   fib(1) fib(0)
```

Notice: `fib(3)` is computed **2 times**, `fib(2)` is computed **3 times**!
This is why naive Fibonacci is exponentially slow.

**Fix:** Use [[Memoization vs Tabulation]] to cache results → O(n) time.

```cpp
int fib(int n, vector<int>& memo) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];           // already computed?
    memo[n] = fib(n-1, memo) + fib(n-2, memo);   // compute & cache
    return memo[n];
}
```

---

### 3. Tree Traversal (Inorder)

```cpp
void inorder(Node* root) {
    if (root == nullptr) return;    // base case: empty tree
    inorder(root->left);           // recurse left subtree
    process(root->val);            // visit current node
    inorder(root->right);          // recurse right subtree
}
```

```
Tree:       4
           / \
          2   6
         / \ / \
        1  3 5  7

Inorder traversal: 1 2 3 4 5 6 7  (sorted!)
```

Time: O(n) | Space: O(h) where h = tree height

---

### 4. Binary Search (Recursive)

```cpp
int binarySearch(int arr[], int lo, int hi, int target) {
    if (lo > hi) return -1;                     // base case: not found
    int mid = lo + (hi - lo) / 2;
    if (arr[mid] == target) return mid;         // base case: found
    if (arr[mid] < target)
        return binarySearch(arr, mid + 1, hi, target);  // search right
    else
        return binarySearch(arr, lo, mid - 1, target);  // search left
}
```

Time: O(log n) | Space: O(log n) call stack

---

### 5. Merge Sort (Divide and Conquer)

```cpp
void mergeSort(int arr[], int lo, int hi) {
    if (lo >= hi) return;                 // base case: 0 or 1 element
    int mid = lo + (hi - lo) / 2;
    mergeSort(arr, lo, mid);              // sort left half
    mergeSort(arr, mid + 1, hi);          // sort right half
    merge(arr, lo, mid, hi);              // merge sorted halves
}
```

```
Divide & Conquer visualization:

    [38, 27, 43, 3, 9, 82, 10]
           /               \
    [38, 27, 43]      [3, 9, 82, 10]       ← DIVIDE
      /      \          /         \
   [38]  [27,43]    [3, 9]    [82, 10]
          / \        / \        /  \
        [27][43]   [3] [9]   [82] [10]     ← BASE CASES
          \ /        \ /        \  /
        [27,43]    [3, 9]    [10, 82]      ← MERGE (conquer)
          \  /        \  /
    [27, 38, 43]   [3, 9, 10, 82]
              \       /
    [3, 9, 10, 27, 38, 43, 82]             ← FINAL MERGE
```

Time: O(n log n) | Space: O(n) for merge buffer + O(log n) stack

---

## 🔹 Time & Space Complexity of Recursive Algorithms

### Recurrence Relations

Recursive algorithms are often described by recurrence relations:

| Algorithm | Recurrence | Solves to |
|-----------|-----------|-----------|
| Factorial | T(n) = T(n-1) + O(1) | O(n) |
| Binary Search | T(n) = T(n/2) + O(1) | O(log n) |
| Merge Sort | T(n) = 2T(n/2) + O(n) | O(n log n) |
| Naive Fibonacci | T(n) = T(n-1) + T(n-2) + O(1) | O(2^n) |

See [[Big O - Definition]] for more on complexity analysis.

### Space Complexity

- **Minimum:** O(max call stack depth)
- **Linear recursion** (one recursive call): O(n) stack depth
- **Binary recursion** (two calls, like merge sort): O(log n) stack depth
- **Exponential recursion** (like naive fib): O(n) stack depth (deepest path)

### The Recursion Tree Method

Draw the tree of all recursive calls. Each node represents work done at that call.

```
For mergeSort on n elements:

Level 0:          [n]                    → O(n) work (one merge of n elements)
                 /   \
Level 1:      [n/2] [n/2]               → O(n) work (two merges of n/2)
              / \    / \
Level 2:   [n/4]×4                       → O(n) work (four merges of n/4)
              ...
Level log(n): [1] × n                    → O(n) work (n base cases)

Total levels: log(n)
Work per level: O(n)
Total: O(n log n)
```

---

## 🔹 Common Pitfalls

### 1. Missing Base Case
```cpp
int bad(int n) {
    return n * bad(n - 1);  // never stops! → Stack Overflow
}
```

### 2. Base Case Doesn't Cover All Stopping Conditions
```cpp
int factorial(int n) {
    if (n == 1) return 1;           // what if n = 0? or negative?
    return n * factorial(n - 1);    // factorial(0) → factorial(-1) → ...overflow
}

// Fix:
int factorial(int n) {
    if (n <= 1) return 1;           // covers 0 and 1
    return n * factorial(n - 1);
}
```

### 3. Not Making the Problem Smaller
```cpp
int infinite(int n) {
    if (n == 0) return 0;
    return infinite(n);     // n never changes! → infinite recursion
}
```

### 4. Redundant Computation
```
Naive fib(50) makes ~2^50 ≈ 10^15 calls.
Memoized fib(50) makes only 50 calls.
```

Always ask: "Am I solving the same subproblem multiple times?" If yes, see [[Memoization vs Tabulation]] and [[Dynamic Programming]].

---

## 🔹 Template for Writing Recursive Functions

```cpp
ReturnType solve(Parameters) {
    // 1. Base case(s) — when to stop
    if (base_condition) {
        return base_value;
    }
    
    // 2. Recursive case — break down the problem
    //    Trust that solve(smaller_problem) returns the correct answer
    auto sub_result = solve(smaller_problem);
    
    // 3. Combine — use sub_result to build the answer for current problem
    return combine(current_data, sub_result);
}
```

### Checklist before writing:
- [ ] What is the base case? (smallest possible input)
- [ ] What is the recursive case? (how to shrink the problem)
- [ ] Am I ALWAYS making the problem smaller?
- [ ] Do my base cases cover ALL stopping conditions?
- [ ] Am I recomputing the same subproblems? (consider memoization)

---

## 🔹 See Also

- [[Backtracking]] — recursion + choosing + undoing choices
- [[Dynamic Programming]] — recursion + memoization for overlapping subproblems
- [[Memoization vs Tabulation]] — top-down vs bottom-up approaches
- [[DFS - Depth First Search]] — graph traversal that naturally uses recursion
- [[Big O - Definition]] — analyzing time and space complexity
