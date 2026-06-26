---
tags:
  - algorithms
  - technique
  - dynamic-programming
---
# Memoization vs Tabulation

## 🔹 Real-World Analogy

**Memoization — remembering answers you already computed:**
Imagine someone keeps asking you "What's 3 + 7?" over and over. The first time, you compute it: 10. Every time after that, you just *remember* it's 10. No recomputation needed. That's memoization — you solve a problem once, write the answer on a sticky note, and look it up next time.

**Tabulation — building from the ground up:**
Imagine building a brick wall. You lay the bottom row first, then the next row on top, then the next. Each row depends on what's already below it. You never try to place a top brick before the foundation exists. That's tabulation — you start from the smallest subproblems and systematically build your way up to the answer.

---

## 🔹 What is Dynamic Programming (DP)?

Dynamic Programming is a technique for solving problems by breaking them into **overlapping subproblems** and solving each subproblem **only once**, storing its result for future use.

### Two Key Properties a Problem MUST Have for DP

**a) Optimal Substructure**
The optimal solution to the problem contains optimal solutions to its subproblems.

```
Example: Shortest path A → D

If shortest path A → D goes through B and C:
   A →(3)→ B →(2)→ C →(4)→ D

Then A → B must also be the shortest path from A to B,
and B → C must be the shortest path from B to C, etc.

The optimal whole = optimal parts
```

**b) Overlapping Subproblems**
The same subproblem is solved multiple times in the recursive solution.

```
Fibonacci:  fib(5)  needs  fib(4) and fib(3)
            fib(4)  needs  fib(3) and fib(2)
                           ^^^^^
                    fib(3) appears TWICE — overlapping!
```

### Decision Tree: When to Use What

```
Does the problem have optimal substructure?
├── NO → Brute force / other technique
└── YES → Does it have overlapping subproblems?
           ├── NO → Divide & Conquer or [[Greedy Technique]]
           └── YES → ★ DYNAMIC PROGRAMMING ★
```

---

## 🔹 The Fibonacci Example — Why DP Matters

### Naive Recursion — The Problem

```cpp
int fib(int n) {
    if (n <= 1) return n;
    return fib(n-1) + fib(n-2);  // two recursive calls!
}
```

The recursion tree explodes with **duplicate work**:

```
                              fib(5)
                           /          \
                       fib(4)          fib(3)       ← fib(3) computed TWICE
                      /     \          /    \
                  fib(3)   fib(2)  fib(2)  fib(1)  ← fib(2) computed 3x
                  /   \     / \     / \
             fib(2) fib(1) f1 f0  f1  f0
              / \
           fib(1) fib(0)
```

```
Call count for fib(5):
  fib(5): 1 time
  fib(4): 1 time
  fib(3): 2 times     ← WASTED
  fib(2): 3 times     ← WASTED
  fib(1): 5 times     ← WASTED
  fib(0): 3 times     ← WASTED
  Total: 15 calls for just fib(5)!
```

**Time: O(2^n) — EXPONENTIAL!** Each level roughly doubles the work.

For fib(50), that's over **1 trillion** calls. Your computer would take *days*.

### With DP: Each Subproblem Solved Only ONCE → O(n)

```
Without DP:  fib(50) ≈ 1,125,899,906,842,624 operations
With DP:     fib(50) ≈ 50 operations

That's a ~22 TRILLION times speedup.
```

==This is the power of DP: turning exponential into polynomial.==

---

## 🔹 Approach 1: Top-Down (Memoization)

**Strategy:** Start from the original problem (the "top") and break it down recursively. Before computing any subproblem, check if you've already solved it. If yes, return the cached answer. If no, compute it, cache it, then return.

```
Think of it as: "Solve recursively, but don't be stupid about it — 
                 remember what you've already computed."
```

### How It Works — Step by Step

```
1. Call fib(5)
2. fib(5) not cached → compute it
3.   Need fib(4) — not cached → compute
4.     Need fib(3) — not cached → compute
5.       Need fib(2) — not cached → compute
6.         Need fib(1) → base case = 1
7.         Need fib(0) → base case = 0
8.       fib(2) = 1 → CACHE IT ✓
9.       Need fib(1) → base case = 1
10.    fib(3) = 2 → CACHE IT ✓
11.    Need fib(2) → CACHED! Return 1 instantly ★
12.  fib(4) = 3 → CACHE IT ✓
13.  Need fib(3) → CACHED! Return 2 instantly ★
14. fib(5) = 5 → CACHE IT ✓
```

### Visualization — Cache Hits vs Misses

```
fib(5) → needs fib(4), fib(3)
  fib(4) → needs fib(3), fib(2)
    fib(3) → needs fib(2), fib(1)
      fib(2) → needs fib(1), fib(0)
        fib(1) = 1        ✓ base case
        fib(0) = 0        ✓ base case
      fib(2) = 1          ✓ CACHED!
      fib(1) = 1          ✓ base case
    fib(3) = 2            ✓ CACHED!   ← saved entire subtree!
  fib(4) = 3              ✓ CACHED!
  fib(3) = 2              ✓ ALREADY CACHED — no recomputation!
fib(5) = 5                ✓ DONE
```

### Complete Code

```cpp
// Top-Down: Memoization
int memo[1001];  // -1 = not computed yet

int fib(int n) {
    if (n <= 1) return n;              // base case
    if (memo[n] != -1) return memo[n]; // already solved? return cached!

    memo[n] = fib(n-1) + fib(n-2);    // solve & store
    return memo[n];
}

// Usage:
// memset(memo, -1, sizeof(memo));
// int answer = fib(n);

// Time:  O(n) — each subproblem computed at most once
// Space: O(n) for memo array + O(n) for recursion call stack
```

### Recursion Stack Visualization

```
Call Stack at deepest point (computing fib(5)):

┌──────────┐
│  fib(1)  │ ← currently executing (base case)
├──────────┤
│  fib(2)  │ ← waiting for fib(1)
├──────────┤
│  fib(3)  │ ← waiting for fib(2)
├──────────┤
│  fib(4)  │ ← waiting for fib(3)
├──────────┤
│  fib(5)  │ ← waiting for fib(4)
└──────────┘
Stack depth = n (risk of stack overflow for large n!)
```

---

## 🔹 Approach 2: Bottom-Up (Tabulation)

**Strategy:** Start from the smallest subproblems (the "bottom") and build up to the answer iteratively. Fill a table in the correct order so that when you need a value, it's already been computed.

```
Think of it as: "Build the answer from the ground up, 
                 like laying bricks for a wall."
```

### How It Works — Step by Step

```
Goal: compute fib(5)

Start with what we know (base cases):
  dp[0] = 0
  dp[1] = 1

Build up iteratively:
  dp[2] = dp[1] + dp[0] = 1 + 0 = 1
  dp[3] = dp[2] + dp[1] = 1 + 1 = 2
  dp[4] = dp[3] + dp[2] = 2 + 1 = 3
  dp[5] = dp[4] + dp[3] = 3 + 2 = 5  ← ANSWER!
```

### Visualization — Table Being Filled

```
Step 0: dp: [0] [1] [ ] [ ] [ ] [ ]
              ↑   ↑
              base cases (given)

Step 1: dp: [0] [1] [1] [ ] [ ] [ ]      i=2: dp[2] = dp[1] + dp[0] = 1
                 └─┘ └─┘
                  ↓    ↓
                  1  +  0  =  1

Step 2: dp: [0] [1] [1] [2] [ ] [ ]      i=3: dp[3] = dp[2] + dp[1] = 2
                      └─┘└─┘
                       ↓   ↓
                       1 + 1 = 2

Step 3: dp: [0] [1] [1] [2] [3] [ ]      i=4: dp[4] = dp[3] + dp[2] = 3
                          └─┘└─┘
                           ↓   ↓
                           2 + 1 = 3

Step 4: dp: [0] [1] [1] [2] [3] [5]      i=5: dp[5] = dp[4] + dp[3] = 5  ★
                              └─┘└─┘
                               ↓   ↓
                               3 + 2 = 5
```

### Complete Code

```cpp
// Bottom-Up: Tabulation
int fib(int n) {
    if (n <= 1) return n;

    int dp[n + 1];
    dp[0] = 0;  // base case
    dp[1] = 1;  // base case

    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i-1] + dp[i-2];  // build from smaller subproblems
    }
    return dp[n];
}

// Time:  O(n)
// Space: O(n) for the dp array
// NO recursion stack — no stack overflow risk!
```

---

## 🔹 Space Optimization — Keep Only What You Need

Key insight: Fibonacci only needs the **last 2 values** to compute the next one. Why store the entire table?

```
Before optimization:  dp: [0] [1] [1] [2] [3] [5] [8] [13] ...
                           ↑   ↑   ↑   ↑   ↑   ↑   ↑    ↑
                           we keep ALL of these (waste!)

After optimization:   prev2  prev1  curr
                        ↓      ↓      ↓
                       [3]   [5]    [8]     ← only 3 variables!
```

### Visualization of the Sliding Window

```
i=2:  prev2=0   prev1=1   → curr = 0+1 = 1
i=3:  prev2=1   prev1=1   → curr = 1+1 = 2
i=4:  prev2=1   prev1=2   → curr = 1+2 = 3
i=5:  prev2=2   prev1=3   → curr = 2+3 = 5  ★

      ┌───┐ ┌───┐
      │ p2│→│ p1│→ curr
      └───┘ └───┘
         slide window forward each iteration
```

### Complete Code

```cpp
// Space-Optimized Bottom-Up
int fib(int n) {
    if (n <= 1) return n;
    int prev2 = 0, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}

// Time:  O(n)
// Space: O(1) ← HUGE improvement from O(n)!
```

---

## 🔹 When to Choose Each Approach

| Aspect | Top-Down (Memoization) | Bottom-Up (Tabulation) |
|---|---|---|
| **Style** | Recursive + cache | Iterative + table |
| **Thinking** | More natural — just add caching to recursion | Need to figure out computation order |
| **Stack space** | O(depth) recursion stack | None |
| **Subproblems solved** | Only those actually needed | ALL subproblems in table |
| **Speed** | Slightly slower (function call overhead) | Slightly faster |
| **Space optimization** | Harder to reduce | Easy to reduce to O(1) when possible |
| **Best when** | Not all subproblems needed; tree-shaped recursion | All subproblems needed; clear left-to-right order |
| **Stack overflow risk** | Yes (deep recursion) | No |
| **Debugging** | Harder (recursive call stack) | Easier (print the table) |

### Quick Decision Guide

```
Should I use Memoization or Tabulation?

Is the recursion depth very large (> ~10,000)?
├── YES → Tabulation (avoid stack overflow)
└── NO
    ├── Do I need ALL subproblems?
    │   ├── YES → Tabulation (more efficient)
    │   └── NO  → Memoization (skip unneeded subproblems)
    └── Am I more comfortable thinking recursively?
        ├── YES → Memoization (start here, optimize later)
        └── NO  → Tabulation
```

> **Pro tip:** In interviews, start with memoization (easier to think about), then convert to tabulation if asked to optimize space.

---

## 🔹 How to Identify DP Problems — Pattern Recognition

Look for these signals in problem statements:

```
Signal Phrases That Scream "DP!":
┌─────────────────────────────────────────────┐
│  "Find the MINIMUM/MAXIMUM..."              │
│  "Count the NUMBER OF WAYS..."              │
│  "Is it POSSIBLE to..."  (yes/no DP)        │
│  "Find the LONGEST/SHORTEST..."             │
│  "What is the OPTIMAL..."                   │
└─────────────────────────────────────────────┘

Structural Signals:
┌─────────────────────────────────────────────┐
│  • Choices at each step                     │
│  • Need the OVERALL optimal, not just       │
│    locally optimal ([[Greedy Technique]])    │
│  • Brute force is exponential               │
│  • Subproblems repeat                       │
└─────────────────────────────────────────────┘
```

### DP vs Greedy — The Key Difference

```
Greedy:  Make the locally best choice at each step
         → Sometimes gives the global optimum (coin change with standard coins)
         → Sometimes FAILS (coin change with arbitrary coins)

DP:      Consider ALL choices at each step, pick the best overall
         → Always gives the global optimum
         → More expensive but always correct
```

---

## 🔹 The 5-Step Framework to Solve ANY DP Problem

==This is the single most important framework for DP. Master it.==

```
┌─────────────────────────────────────────────────────────┐
│  STEP 1: Define the STATE                               │
│          What does dp[i] (or dp[i][j]) represent?       │
│          ★ This is the HARDEST and most important step  │
│                                                         │
│  STEP 2: Find the RECURRENCE RELATION                   │
│          How does dp[i] relate to smaller subproblems?   │
│                                                         │
│  STEP 3: Identify BASE CASES                            │
│          What are the smallest subproblems you know?     │
│                                                         │
│  STEP 4: Determine COMPUTATION ORDER                    │
│          Fill the table so dependencies are ready first  │
│                                                         │
│  STEP 5: IMPLEMENT                                      │
│          Choose memo or tabulation, code it up           │
└─────────────────────────────────────────────────────────┘
```

### Concrete Example: Climbing Stairs

**Problem:** You have `n` stairs. You can climb 1 or 2 steps at a time. How many distinct ways can you reach the top?

```
n=4 example — all possible ways:

  Way 1: 1+1+1+1   (step step step step)
  Way 2: 1+1+2     (step step jump)
  Way 3: 1+2+1     (step jump step)
  Way 4: 2+1+1     (jump step step)
  Way 5: 2+2       (jump jump)

  Answer: 5 ways
```

**Step 1: Define the State**
```
dp[i] = number of distinct ways to reach step i
```

**Step 2: Find the Recurrence**
To reach step `i`, you came from either step `i-1` (took 1 step) or step `i-2` (took 2 steps):
```
dp[i] = dp[i-1] + dp[i-2]

                     step i
                    /       \
               came from     came from
              step (i-1)    step (i-2)
              via 1 step    via 2 steps
```

**Step 3: Base Cases**
```
dp[0] = 1    (1 way to stay at ground: do nothing)
dp[1] = 1    (1 way to reach step 1: take 1 step)
```

**Step 4: Computation Order**
Left to right: dp[0] → dp[1] → dp[2] → ... → dp[n]

**Step 5: Implement**

```cpp
int climbStairs(int n) {
    if (n <= 1) return 1;

    int dp[n + 1];
    dp[0] = 1;
    dp[1] = 1;

    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i-1] + dp[i-2];
    }
    return dp[n];
}

// Time: O(n), Space: O(n)
// Can optimize to O(1) space — same as Fibonacci!
```

### Verification — Building the Table

```
dp[0] = 1
dp[1] = 1
dp[2] = dp[1] + dp[0] = 1 + 1 = 2    (ways: 1+1, 2)
dp[3] = dp[2] + dp[1] = 2 + 1 = 3    (ways: 1+1+1, 1+2, 2+1)
dp[4] = dp[3] + dp[2] = 3 + 2 = 5    (ways: 1+1+1+1, 1+1+2, 1+2+1, 2+1+1, 2+2)  ✓
```

---

## 🔹 Common Pitfalls

| Pitfall | What Goes Wrong | How to Avoid |
|---|---|---|
| **Wrong state definition** | dp[i] doesn't capture enough information to make decisions | Ask: "With just dp[i], can I compute dp[i+1]?" |
| **Missing base cases** | Array out of bounds or wrong answers for small inputs | Always test with n=0, n=1 manually |
| **Wrong computation order** | Using dp[j] before it's been computed | Draw dependency arrows; fill in dependency order |
| **No space optimization** | Using O(n) or O(n^2) space when O(1) or O(n) suffices | After solving, ask: "How much past state do I actually need?" |
| **Using DP when greedy works** | Overcomplicating simple problems | Try greedy first; if it fails on edge cases, then use DP |
| **Off-by-one errors** | dp array too small, or i starts/ends at wrong value | Be explicit about what dp[0] and dp[n] mean |

---

## 🔹 Summary

```
┌─────────────────────────────────────────────────────────────┐
│                   DYNAMIC PROGRAMMING                       │
│                                                             │
│  Core Idea: Don't solve the same subproblem twice!          │
│                                                             │
│  ┌─────────────────┐     ┌──────────────────────┐          │
│  │   MEMOIZATION    │     │    TABULATION         │          │
│  │   (Top-Down)     │     │    (Bottom-Up)        │          │
│  │                  │     │                       │          │
│  │  Recursive       │     │  Iterative            │          │
│  │  + Cache         │     │  + Table              │          │
│  │                  │     │                       │          │
│  │  Natural to      │     │  Easy to optimize     │          │
│  │  think about     │     │  space                │          │
│  │                  │     │                       │          │
│  │  Stack overflow  │     │  No stack risk        │          │
│  │  risk            │     │                       │          │
│  └─────────────────┘     └──────────────────────┘          │
│                                                             │
│  5-Step Framework:                                          │
│  1. Define state    → What does dp[i] mean?                 │
│  2. Recurrence      → dp[i] = f(smaller subproblems)        │
│  3. Base cases      → What do we know for free?             │
│  4. Order           → Fill dependencies first               │
│  5. Implement       → Code it!                              │
│                                                             │
│  Power: Exponential O(2^n) → Polynomial O(n) or O(n^2)     │
└─────────────────────────────────────────────────────────────┘
```

---

**See also:** [[Common DP Patterns]] | [[Recursion]] | [[Backtracking]] | [[Greedy Technique]] | [[Big O - Definition]]
