---
tags:
  - algorithms
  - pattern-recognition
  - dynamic-programming
---

## 🔹 When to Suspect This Pattern

- **Optimization** keywords: "minimum cost", "maximum profit", "number of ways", "longest/shortest"
- Problem asks "**can you reach...?**" or "**is it possible?**"
- The problem has **optimal substructure** — optimal solution uses optimal solutions to subproblems
- The problem has **overlapping subproblems** — same subproblem computed multiple times
- Brute force would be exponential (2^n or n!) — DP brings it to polynomial
- Keywords: "coin change", "knapsack", "edit distance", "longest subsequence", "count paths"

---

## 🔹 Confirming It's the Right Pattern

The **3-question test**:

1. **Can I make a choice at each step?** (take item or skip, move right or down, use this coin or not)
2. **Does each choice create a smaller subproblem of the same type?**
3. **Do subproblems overlap?** (Same subproblem appears in multiple branches of recursion tree)

If all three are YES → **DP**. If 1 and 2 but NOT 3 → consider [[Pattern - Greedy]] or divide-and-conquer.

> [!warning] DP vs Greedy
> Both solve optimization problems. The difference:
> - **Greedy**: make the locally best choice, never reconsider → O(n) or O(n log n)
> - **DP**: try all choices, pick the best → higher complexity but always correct
>
> If you can't prove the greedy choice is always optimal, use DP.

---

## 🔹 The DP Framework (5 Steps)

1. **Define the state** — what does `dp[i]` (or `dp[i][j]`) represent?
2. **Find the recurrence** — how does `dp[i]` relate to smaller subproblems?
3. **Identify base cases** — what are the trivial answers? (`dp[0]`, `dp[0][0]`, etc.)
4. **Determine traversal order** — fill table so dependencies are computed first
5. **Optimize space if possible** — often only need previous row/column

---

## 🔹 Common DP Categories

| Category | Example Problems | State Pattern |
|---|---|---|
| **Linear DP** | Climbing Stairs, House Robber, LIS | `dp[i]` = answer for first i elements |
| **Two-Sequence DP** | Edit Distance, LCS | `dp[i][j]` = answer for first i of A, first j of B |
| **Knapsack** | 0/1 Knapsack, Coin Change, Partition | `dp[i][w]` = answer using first i items with capacity w |
| **Grid DP** | Unique Paths, Min Path Sum | `dp[i][j]` = answer to reach cell (i, j) |
| **Interval DP** | Matrix Chain, Burst Balloons | `dp[i][j]` = answer for subarray [i..j] |
| **Tree DP** | House Robber III, Diameter | `dp[node]` = answer for subtree rooted at node |
| **Bitmask DP** | TSP, Assign Tasks | `dp[mask]` = answer for subset represented by bitmask |
| **String DP** | Palindrome Partitioning, Word Break | `dp[i]` or `dp[i][j]` on string indices |

---

## 🔹 Template: Bottom-Up (Tabulation)

```cpp
// Coin Change: minimum coins to make amount
vector<int> dp(amount + 1, INT_MAX);
dp[0] = 0;  // base case: 0 coins for amount 0

for (int i = 1; i <= amount; i++) {
    for (int coin : coins) {
        if (coin <= i && dp[i - coin] != INT_MAX)
            dp[i] = min(dp[i], dp[i - coin] + 1);
    }
}
return dp[amount] == INT_MAX ? -1 : dp[amount];
```

### Template: Top-Down (Memoization)

```cpp
unordered_map<int, int> memo;

int solve(int amount, vector<int>& coins) {
    if (amount == 0) return 0;
    if (amount < 0) return INT_MAX;
    if (memo.count(amount)) return memo[amount];

    int result = INT_MAX;
    for (int coin : coins) {
        int sub = solve(amount - coin, coins);
        if (sub != INT_MAX)
            result = min(result, sub + 1);
    }
    return memo[amount] = result;
}
```

> [!tip] Top-Down vs Bottom-Up
> - **Top-down** (memoization): easier to write, natural recursion. Good for problems where not all states are visited
> - **Bottom-up** (tabulation): slightly faster (no recursion overhead), easier to optimize space. Required when recursion depth is too large (stack overflow)

---

## 🔹 Classic Problems

| Problem | State | Recurrence |
|---|---|---|
| **Climbing Stairs** | `dp[i]` = ways to reach step i | `dp[i] = dp[i-1] + dp[i-2]` |
| **Coin Change** | `dp[i]` = min coins for amount i | `dp[i] = min(dp[i - coin] + 1)` |
| **0/1 Knapsack** | `dp[i][w]` = max value, i items, cap w | `dp[i][w] = max(skip, take)` |
| **Longest Common Subsequence** | `dp[i][j]` = LCS of A[0..i], B[0..j] | Match → `dp[i-1][j-1]+1`, else `max(dp[i-1][j], dp[i][j-1])` |
| **Edit Distance** | `dp[i][j]` = min edits A[0..i] → B[0..j] | Insert, delete, replace — take min |
| **Longest Increasing Subseq** | `dp[i]` = LIS ending at i | `dp[i] = max(dp[j] + 1)` for j < i where A[j] < A[i] |
| **Unique Paths** | `dp[i][j]` = paths to (i, j) | `dp[i][j] = dp[i-1][j] + dp[i][j-1]` |
| **House Robber** | `dp[i]` = max money from first i houses | `dp[i] = max(dp[i-1], dp[i-2] + val[i])` |
| **Word Break** | `dp[i]` = can we segment s[0..i]? | `dp[i] = any dp[j] && s[j..i] in dict` |

---

## 🔹 Space Optimization Patterns

| Original | Optimized | When |
|---|---|---|
| `dp[n]` (1D) | Two variables | Only depends on `dp[i-1]`, `dp[i-2]` |
| `dp[n][m]` (2D) | `dp[m]` (1D) | Only depends on previous row |
| `dp[n][W]` (knapsack) | `dp[W]` (1D) | Iterate W backwards for 0/1 knapsack |

---

## 🔹 Common Mistakes

- **Wrong state definition**: if `dp[i]` isn't clearly defined, the recurrence will be wrong. Write the definition in a comment first
- **Missing base cases**: forgetting `dp[0] = 0` or `dp[0][0] = true` leads to garbage results
- **Wrong traversal order**: dependencies must be computed before current cell. Draw the dependency arrows
- **0/1 vs unbounded knapsack**: 0/1 = iterate capacity backwards; unbounded = forwards
- **Confusing subsequence vs subarray**: subsequence = non-contiguous (DP); subarray = contiguous (often sliding window)
- **INT_MAX + 1 overflow**: when using `INT_MAX` as "impossible", always check before adding to it

---

## 🔹 Related Patterns

- [[Pattern - Greedy]] — if greedy choice property holds, prefer greedy (simpler)
- [[Pattern - Backtracking]] — DP = backtracking + memoization
- [[Pattern - Binary Search]] — LIS can be solved in O(n log n) with BS + DP
- [[Pattern - Graph]] — shortest path algorithms are graph DP
- [[Constraint Analysis]] — DP state space determines feasibility (n^2 states OK if n <= 5000)
