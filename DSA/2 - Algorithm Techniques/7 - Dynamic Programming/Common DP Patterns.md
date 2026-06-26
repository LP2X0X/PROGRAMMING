---
tags:
  - algorithms
  - technique
  - dynamic-programming
created: 2026-06-25
---

# Common DP Patterns

Dynamic programming solves problems by breaking them into overlapping subproblems,
solving each once, and storing results. Every DP problem follows a pattern -- learn
the patterns and you can solve the vast majority of DP problems by recognizing
which one applies.

**The 80/20**: master 0/1 Knapsack, LCS, and Grid Paths. They cover the majority
of DP problems you will encounter. The rest are variations on the same ideas.

See also: [[Memoization vs Tabulation]] | [[Recursion]] | [[Backtracking]] | [[Greedy Technique]]

---

## 🔹 Pattern 1: 0/1 Knapsack

**Problem**: given n items each with a weight and value, pick items to maximize
total value without exceeding capacity W. Each item is either taken or left (0/1).

**State**: `dp[i][w]` = max value achievable using first i items with capacity w

**Recurrence**:

```
dp[i][w] = max(
    dp[i-1][w],                        // SKIP item i
    dp[i-1][w - wt[i]] + val[i]        // TAKE item i  (only if wt[i] <= w)
)
```

**Base case**: `dp[0][w] = 0` for all w (no items = no value)

### Complete 2D Table

Items: {wt:1, val:1}, {wt:3, val:4}, {wt:4, val:5}, {wt:5, val:7}
Capacity W = 7

```
                         Capacity w
                0    1    2    3    4    5    6    7
              +----+----+----+----+----+----+----+----+
  no items    |  0 |  0 |  0 |  0 |  0 |  0 |  0 |  0 |
              +----+----+----+----+----+----+----+----+
  item 1      |  0 |  1 |  1 |  1 |  1 |  1 |  1 |  1 |
  wt=1, v=1   |    | ^T |    |    |    |    |    |    |
              +----+----+----+----+----+----+----+----+
  item 2      |  0 |  1 |  1 |  4 |  5 |  5 |  5 |  5 |
  wt=3, v=4   |    |    |    | ^T | ^T |    |    |    |
              +----+----+----+----+----+----+----+----+
  item 3      |  0 |  1 |  1 |  4 |  5 |  6 |  6 |  9 |
  wt=4, v=5   |    |    |    |    |    | ^T |    | ^T |
              +----+----+----+----+----+----+----+----+
  item 4      |  0 |  1 |  1 |  4 |  5 |  7 |  8 |  9 |
  wt=5, v=7   |    |    |    |    |    | ^T | ^T |    |
              +----+----+----+----+----+----+----+----+

  ^T = this cell chose to TAKE the item  (blank = SKIP)

  Answer: dp[4][7] = 9
```

How to verify dp[3][7] = 9:
```
  dp[3][7]: take item 3?
    SKIP: dp[2][7] = 5
    TAKE: dp[2][7-4] + 5 = dp[2][3] + 5 = 4 + 5 = 9
    max(5, 9) = 9  --> TAKE

  Backtrack to find which items:
    dp[4][7]=9, dp[3][7]=9  --> item 4 skipped (same value without it)
    dp[3][7]=9, dp[2][3]=4  --> item 3 taken (9 != 5, came from dp[2][3]+5)
    dp[2][3]=4, dp[1][0]=0  --> item 2 taken (4 != 1, came from dp[1][0]+4)
    dp[1][0]=0              --> item 1 skipped (capacity exhausted)

  Items chosen: {2, 3}  weight=3+4=7, value=4+5=9
```

### C++ Code: 2D Table

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int knapsack01(vector<int>& wt, vector<int>& val, int W) {
    int n = wt.size();
    // dp[i][w] = max value using first i items with capacity w
    vector<vector<int>> dp(n + 1, vector<int>(W + 1, 0));

    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            dp[i][w] = dp[i-1][w];  // skip item i
            if (wt[i-1] <= w) {
                dp[i][w] = max(dp[i][w],
                               dp[i-1][w - wt[i-1]] + val[i-1]);
            }
        }
    }
    return dp[n][W];
}

int main() {
    vector<int> wt  = {1, 3, 4, 5};
    vector<int> val = {1, 4, 5, 7};
    int W = 7;
    cout << "Max value: " << knapsack01(wt, val, W) << endl;  // 9
    return 0;
}
```

### C++ Code: Space-Optimized 1D

Key insight: each row only depends on the **previous** row. Use a single 1D
array and iterate capacity **right to left** so we don't overwrite values we
still need.

```
  Why right-to-left?

  When computing dp[w], we need dp[w - wt[i]] from the PREVIOUS row.
  If we go left-to-right, dp[w - wt[i]] already holds THIS row's value.
  Right-to-left guarantees dp[w - wt[i]] still has the previous row's value.

  Iteration direction:
    0/1 Knapsack   --> RIGHT to LEFT  (each item used at most once)
    Unbounded      --> LEFT to RIGHT  (items can be reused)
```

```cpp
int knapsack01_optimized(vector<int>& wt, vector<int>& val, int W) {
    int n = wt.size();
    vector<int> dp(W + 1, 0);

    for (int i = 0; i < n; i++) {
        for (int w = W; w >= wt[i]; w--) {  // RIGHT to LEFT
            dp[w] = max(dp[w], dp[w - wt[i]] + val[i]);
        }
    }
    return dp[W];
}
```

### Variants

- **Subset Sum**: can you pick items whose weights sum exactly to target?
  - `dp[i][s] = true/false`
  - Recurrence: `dp[i][s] = dp[i-1][s] || dp[i-1][s - wt[i]]`
- **Equal Partition**: split array into two subsets with equal sum
  - Reduce to Subset Sum with target = totalSum / 2

---

## 🔹 Pattern 2: Unbounded Knapsack / Coin Change

**Key difference from 0/1**: each item can be used **unlimited** times.
In the 1D array, iterate **left to right** (opposite of 0/1) because reusing
the current row's value means we allow repeated selection of the same item.

### Problem: Coin Change (Minimum Coins)

Given coin denominations and an amount, find the minimum number of coins needed.

**State**: `dp[a]` = minimum coins to make amount a

**Recurrence**:
```
dp[a] = min(dp[a - coin] + 1)  for each coin where coin <= a
```

**Base case**: `dp[0] = 0` (zero coins for amount zero)

### Complete dp Table: coins=[1,3,4], amount=6

```
  amount:   0     1     2     3     4     5     6
          +-----+-----+-----+-----+-----+-----+-----+
  dp:     |  0  |  1  |  2  |  1  |  1  |  2  |  2  |
          +-----+-----+-----+-----+-----+-----+-----+
  coin
  used:     -     1     1     3     4    1+4   3+3

  Step-by-step computation:

  dp[0] = 0                                (base case)

  dp[1]: try coin 1: dp[0]+1 = 1
         try coin 3: 1 < 3, skip
         try coin 4: 1 < 4, skip
         dp[1] = 1                         use [1]

  dp[2]: try coin 1: dp[1]+1 = 2
         try coin 3: 2 < 3, skip
         try coin 4: 2 < 4, skip
         dp[2] = 2                         use [1,1]

  dp[3]: try coin 1: dp[2]+1 = 3
         try coin 3: dp[0]+1 = 1  <-- BETTER
         try coin 4: 3 < 4, skip
         dp[3] = 1                         use [3]

  dp[4]: try coin 1: dp[3]+1 = 2
         try coin 3: dp[1]+1 = 2
         try coin 4: dp[0]+1 = 1  <-- BEST
         dp[4] = 1                         use [4]

  dp[5]: try coin 1: dp[4]+1 = 2  <-- BEST (tie)
         try coin 3: dp[2]+1 = 3
         try coin 4: dp[1]+1 = 2  <-- BEST (tie)
         dp[5] = 2                         use [1,4] or [3,?]

  dp[6]: try coin 1: dp[5]+1 = 3
         try coin 3: dp[3]+1 = 2  <-- BEST
         try coin 4: dp[2]+1 = 3
         dp[6] = 2                         use [3,3]

  Answer: dp[6] = 2 coins  (3 + 3)
```

### C++ Code: Minimum Coins

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <climits>
using namespace std;

int coinChange(vector<int>& coins, int amount) {
    // dp[a] = min coins to make amount a, INT_MAX means impossible
    vector<int> dp(amount + 1, INT_MAX);
    dp[0] = 0;

    for (int a = 1; a <= amount; a++) {
        for (int coin : coins) {
            if (coin <= a && dp[a - coin] != INT_MAX) {
                dp[a] = min(dp[a], dp[a - coin] + 1);
            }
        }
    }
    return dp[amount] == INT_MAX ? -1 : dp[amount];
}

int main() {
    vector<int> coins = {1, 3, 4};
    int amount = 6;
    cout << "Min coins: " << coinChange(coins, amount) << endl;  // 2
    return 0;
}
```

### Variant: Coin Change 2 (Count Number of WAYS)

**Problem**: how many distinct combinations of coins sum to the amount?

**Critical detail**: iterate coins in the outer loop, amount in the inner loop.
This avoids counting permutations -- order does not matter.

```
  Why outer=coins?

  If outer=amount, inner=coins:
    We'd count [1,3] and [3,1] as different ways.

  If outer=coins, inner=amount:
    We process all uses of coin[0] first, then coin[1], etc.
    Forces an ordering, so each combination is counted exactly once.
```

```cpp
int coinChange2(vector<int>& coins, int amount) {
    // dp[a] = number of combinations that sum to amount a
    vector<int> dp(amount + 1, 0);
    dp[0] = 1;  // one way to make 0: use no coins

    for (int coin : coins) {                    // outer: each coin type
        for (int a = coin; a <= amount; a++) {  // inner: amounts
            dp[a] += dp[a - coin];
        }
    }
    return dp[amount];
}

// Example: coins=[1,2,5], amount=5
// Ways: [1,1,1,1,1], [1,1,1,2], [1,2,2], [1,1,3]... wait, no 3-coin.
// Actually: [5], [2,2,1], [2,1,1,1], [1,1,1,1,1] = 4 ways
```

---

## 🔹 Pattern 3: Longest Common Subsequence (LCS)

**Problem**: find the longest subsequence common to two strings.
A subsequence keeps characters in order but can skip characters (not contiguous).

**State**: `dp[i][j]` = LCS length of s1[0..i-1] and s2[0..j-1]

**Recurrence**:
```
If s1[i-1] == s2[j-1]:
    dp[i][j] = dp[i-1][j-1] + 1       // characters match, extend LCS

Else:
    dp[i][j] = max(dp[i-1][j],         // skip char from s1
                   dp[i][j-1])          // skip char from s2
```

**Base case**: `dp[0][j] = 0`, `dp[i][0] = 0` (empty string has LCS 0)

### Complete 2D Table: s1="ABCDE", s2="ACE"

```
            ""    A     C     E
          +-----+-----+-----+-----+
  ""      |  0  |  0  |  0  |  0  |
          +-----+-----+-----+-----+
  A       |  0  |  1  |  1  |  1  |   A==A: dp[0][0]+1 = 1
          |     |  \  |  <  |  <  |
          +-----+-----+-----+-----+
  B       |  0  |  1  |  1  |  1  |   B!=A,C,E: max of neighbors
          |     |  ^  |  \  |  <  |
          +-----+-----+-----+-----+
  C       |  0  |  1  |  2  |  2  |   C==C: dp[1][1]+1 = 2
          |     |  ^  |  \  |  <  |
          +-----+-----+-----+-----+
  D       |  0  |  1  |  2  |  2  |   D!=A,C,E: max of neighbors
          |     |  ^  |  ^  |  ^  |
          +-----+-----+-----+-----+
  E       |  0  |  1  |  2  |  3  |   E==E: dp[3][2]+1 = 3
          |     |  ^  |  ^  |  \  |
          +-----+-----+-----+-----+

  Arrows:  \ = diagonal (MATCH)   ^ = up (skip s1 char)   < = left (skip s2 char)

  Answer: dp[5][3] = 3
```

Backtracking to reconstruct the LCS:

```
  Start at dp[5][3] = 3

  dp[5][3]: s1[4]='E' == s2[2]='E'  --> MATCH, record 'E', go diagonal to dp[4][2]
  dp[4][2]: s1[3]='D' != s2[1]='C'  --> dp[3][2]=2 > dp[4][1]=1, go UP to dp[3][2]
  dp[3][2]: s1[2]='C' == s2[1]='C'  --> MATCH, record 'C', go diagonal to dp[2][1]
  dp[2][1]: s1[1]='B' != s2[0]='A'  --> dp[1][1]=1 >= dp[2][0]=0, go UP to dp[1][1]
  dp[1][1]: s1[0]='A' == s2[0]='A'  --> MATCH, record 'A', go diagonal to dp[0][0]
  dp[0][0]: done (reached boundary)

  Collected in reverse: E, C, A  -->  reversed: "ACE"
```

### C++ Code: LCS with Reconstruction

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

string longestCommonSubsequence(const string& s1, const string& s2) {
    int m = s1.size(), n = s2.size();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

    // Fill the table
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1[i-1] == s2[j-1]) {
                dp[i][j] = dp[i-1][j-1] + 1;
            } else {
                dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
            }
        }
    }

    // Reconstruct the LCS by backtracking through the table
    string lcs;
    int i = m, j = n;
    while (i > 0 && j > 0) {
        if (s1[i-1] == s2[j-1]) {
            lcs += s1[i-1];    // this char is in the LCS
            i--; j--;          // move diagonally
        } else if (dp[i-1][j] > dp[i][j-1]) {
            i--;               // move up (skip s1 char)
        } else {
            j--;               // move left (skip s2 char)
        }
    }
    reverse(lcs.begin(), lcs.end());  // we collected chars backwards
    return lcs;
}

int main() {
    string s1 = "ABCDE", s2 = "ACE";
    string result = longestCommonSubsequence(s1, s2);
    cout << "LCS: \"" << result
         << "\" (length " << result.size() << ")" << endl;
    // LCS: "ACE" (length 3)
    return 0;
}
```

---

## 🔹 Pattern 4: Longest Increasing Subsequence (LIS)

**Problem**: find the length of the longest strictly increasing subsequence.

**State**: `dp[i]` = length of LIS **ending at** index i

**Recurrence**:
```
dp[i] = max(dp[j] + 1)  for all j < i where arr[j] < arr[i]
dp[i] = 1               if no such j exists (element alone)
```

### Complete dp Array: arr = [10, 9, 2, 5, 3, 7, 101, 18]

```
  Index:   0    1    2    3    4    5     6     7
  arr:    10    9    2    5    3    7   101    18
  dp:      1    1    1    2    2    3     4     4

  +-------+----+-----------------------------------------------+
  | arr[i]| dp | Reasoning                                     |
  +-------+----+-----------------------------------------------+
  |  10   |  1 | No previous element < 10  (start)             |
  |   9   |  1 | No previous element < 9  (10 >= 9, skip)      |
  |   2   |  1 | No previous element < 2                       |
  |   5   |  2 | j=2: arr[2]=2 < 5  --> dp[2]+1 = 2            |
  |   3   |  2 | j=2: arr[2]=2 < 3  --> dp[2]+1 = 2            |
  |   7   |  3 | j=3: arr[3]=5 < 7  --> dp[3]+1 = 3  (best)    |
  |       |    | j=4: arr[4]=3 < 7  --> dp[4]+1 = 3  (tie)     |
  | 101   |  4 | j=5: arr[5]=7 < 101 --> dp[5]+1 = 4  (best)   |
  |  18   |  4 | j=5: arr[5]=7 < 18  --> dp[5]+1 = 4  (best)   |
  +-------+----+-----------------------------------------------+

  Answer: max(dp) = 4

  Possible LIS of length 4:
    [2, 5, 7, 101]
    [2, 3, 7, 101]
    [2, 5, 7, 18]
    [2, 3, 7, 18]
```

### C++ Code: O(n^2)

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int longestIncreasingSubsequence(vector<int>& arr) {
    int n = arr.size();
    if (n == 0) return 0;

    vector<int> dp(n, 1);  // every element is an LIS of length 1

    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (arr[j] < arr[i]) {
                dp[i] = max(dp[i], dp[j] + 1);
            }
        }
    }
    return *max_element(dp.begin(), dp.end());
}

int main() {
    vector<int> arr = {10, 9, 2, 5, 3, 7, 101, 18};
    cout << "LIS length: "
         << longestIncreasingSubsequence(arr) << endl;  // 4
    return 0;
}
```

### O(n log n) Approach: Patience Sorting / Binary Search

Maintain a `tails` array where `tails[i]` = smallest tail element of all
increasing subsequences of length i+1. For each element:
- If larger than all tails: extend the list (new longest LIS found).
- Otherwise: binary search for the first tail >= element and replace it.

The length of `tails` at the end equals the LIS length.

```
  arr:  10   9   2   5   3   7  101  18

  tails after processing each element:
    Process 10:  [10]                   start
    Process  9:  [9]                    9 replaces 10 (9 < 10)
    Process  2:  [2]                    2 replaces 9
    Process  5:  [2, 5]                 5 > all, extend
    Process  3:  [2, 3]                 3 replaces 5
    Process  7:  [2, 3, 7]             7 > all, extend
    Process 101: [2, 3, 7, 101]        101 > all, extend
    Process  18: [2, 3, 7, 18]         18 replaces 101

  Final length of tails = 4 = LIS length

  NOTE: tails itself is NOT necessarily a valid LIS!
        It only tracks the smallest possible tail for each length.
```

```cpp
int lisNLogN(vector<int>& arr) {
    vector<int> tails;
    for (int x : arr) {
        // Find first element in tails >= x
        auto it = lower_bound(tails.begin(), tails.end(), x);
        if (it == tails.end()) {
            tails.push_back(x);   // extend: new longest subsequence
        } else {
            *it = x;              // replace: keep tails as small as possible
        }
    }
    return tails.size();  // length of tails = LIS length
}
// Time: O(n log n)   Space: O(n)
```

---

## 🔹 Pattern 5: Grid Paths

### Problem 1: Unique Paths

Count unique paths from top-left (0,0) to bottom-right (m-1, n-1).
You can only move **right** or **down**.

**State**: `dp[i][j]` = number of paths to reach cell (i, j)

**Recurrence**: `dp[i][j] = dp[i-1][j] + dp[i][j-1]`

**Base case**: `dp[0][j] = 1`, `dp[i][0] = 1` (only one way along edges)

### Grid Visualization (3 rows x 4 cols)

```
  +------+------+------+------+
  | [1]  --> [1] --> [1] --> [1] |
  +--+---+--+---+--+---+--+---+
     |       |       |       |
     v       v       v       v
  +------+------+------+------+
  | [1]  --> [2] --> [3] --> [4] |
  +--+---+--+---+--+---+--+---+
     |       |       |       |
     v       v       v       v
  +------+------+------+------+
  | [1]  --> [3] --> [6] --> [10]|
  +------+------+------+------+

  How values are computed:
    dp[0][*] = 1          (top row: only way is -->-->-->)
    dp[*][0] = 1          (left col: only way is down)
    dp[1][1] = 1 + 1 = 2  (from above + from left)
    dp[1][2] = 1 + 2 = 3
    dp[1][3] = 1 + 3 = 4
    dp[2][1] = 1 + 2 = 3
    dp[2][2] = 3 + 3 = 6
    dp[2][3] = 4 + 6 = 10

  Answer: dp[2][3] = 10 unique paths
```

### C++ Code: Unique Paths

```cpp
#include <iostream>
#include <vector>
using namespace std;

int uniquePaths(int m, int n) {
    vector<vector<int>> dp(m, vector<int>(n, 1));
    // First row and first column are already 1

    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = dp[i-1][j] + dp[i][j-1];
        }
    }
    return dp[m-1][n-1];
}

int main() {
    cout << "Unique paths (3x4): " << uniquePaths(3, 4) << endl;  // 10
    return 0;
}
```

### Problem 2: Minimum Path Sum

Given a grid with non-negative numbers, find a path from top-left to
bottom-right that minimizes the sum along the path.

**Recurrence**: `dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])`

```
  Grid:                  dp (min path sum):
  +-----+-----+-----+   +-----+-----+-----+
  |  1  |  3  |  1  |   |  1  |  4  |  5  |
  +-----+-----+-----+   +-----+-----+-----+
  |  1  |  5  |  1  |   |  2  |  7  |  6  |
  +-----+-----+-----+   +-----+-----+-----+
  |  4  |  2  |  1  |   |  6  |  8  | [7] |  <-- answer
  +-----+-----+-----+   +-----+-----+-----+

  dp[0][0] = 1
  dp[0][1] = 1 + 3 = 4
  dp[0][2] = 4 + 1 = 5
  dp[1][0] = 1 + 1 = 2
  dp[1][1] = 5 + min(4, 2) = 5 + 2 = 7
  dp[1][2] = 1 + min(5, 7) = 1 + 5 = 6
  dp[2][0] = 2 + 4 = 6
  dp[2][1] = 2 + min(7, 6) = 2 + 6 = 8
  dp[2][2] = 1 + min(6, 8) = 1 + 6 = 7

  Optimal path: 1 -> 3 -> 1 -> 1 -> 1 = 7
                (right, right, down, down)
```

### C++ Code: Minimum Path Sum

```cpp
int minPathSum(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    vector<vector<int>> dp(m, vector<int>(n, 0));

    dp[0][0] = grid[0][0];

    // First row: can only come from the left
    for (int j = 1; j < n; j++)
        dp[0][j] = dp[0][j-1] + grid[0][j];

    // First column: can only come from above
    for (int i = 1; i < m; i++)
        dp[i][0] = dp[i-1][0] + grid[i][0];

    // Fill rest
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1]);
        }
    }
    return dp[m-1][n-1];
}
```

---

## 🔹 Pattern 6: String DP -- Edit Distance

**Problem**: given two words, find the minimum number of operations to transform
word1 into word2. Allowed operations: **insert**, **delete**, **replace**.

**State**: `dp[i][j]` = edit distance between word1[0..i-1] and word2[0..j-1]

**Recurrence**:
```
If word1[i-1] == word2[j-1]:
    dp[i][j] = dp[i-1][j-1]           // characters match, no op needed

Else:
    dp[i][j] = 1 + min(
        dp[i-1][j],                    // DELETE  from word1
        dp[i][j-1],                    // INSERT  into word1
        dp[i-1][j-1]                   // REPLACE in word1
    )
```

**Base cases**:
- `dp[i][0] = i` (delete all chars from word1)
- `dp[0][j] = j` (insert all chars of word2)

### Complete 2D Table: "horse" -> "ros"

```
              ""     r     o     s
            +-----+-----+-----+-----+
    ""      |  0  |  1  |  2  |  3  |   <-- insert r, insert o, insert s
            +-----+-----+-----+-----+
    h       |  1  |  1  |  2  |  3  |
            +-----+-----+-----+-----+
    o       |  2  |  2  |  1  |  2  |
            +-----+-----+-----+-----+
    r       |  3  |  2  |  2  |  2  |
            +-----+-----+-----+-----+
    s       |  4  |  3  |  3  |  2  |
            +-----+-----+-----+-----+
    e       |  5  |  4  |  4  |  3  |
            +-----+-----+-----+-----+

  Answer: dp[5][3] = 3
```

Detailed computation for key cells:

```
  dp[1][1] (h vs r):
    h != r, so: delete  dp[0][1] = 1
                insert  dp[1][0] = 1
                replace dp[0][0] = 0
    dp[1][1] = 1 + min(1, 1, 0) = 1  (REPLACE h->r)

  dp[2][2] (ho vs ro):
    o == o --> dp[1][1] = 1  (MATCH, free!)

  dp[3][1] (hor vs r):
    r == r --> dp[2][0] = 2  (MATCH, free!)

  dp[4][3] (hors vs ros):
    s == s --> dp[3][2] = 2  (MATCH, free!)

  dp[5][3] (horse vs ros):
    e != s, so: delete  dp[4][3] = 2
                insert  dp[5][2] = 4
                replace dp[4][2] = 3 -- wait, let me recheck
    ...actually: 1 + min(dp[4][3], dp[5][2], dp[4][2])
               = 1 + min(2, 4, 3) = 3  (DELETE e from word1)

  Operation map at each non-base cell:
  +--------+--------+--------+
  | REP    | INS o  | INS s  |
  | h->r   |        |        |
  +--------+--------+--------+
  | DEL o  | MATCH  | INS s  |
  |        | o==o   |        |
  +--------+--------+--------+
  | DEL r  | MATCH  | DEL/REP|
  |        | r==r   |        |
  +--------+--------+--------+
  | DEL s  | DEL    | MATCH  |
  |        |        | s==s   |
  +--------+--------+--------+
  | DEL e  | DEL    | DEL    |
  |        |        | e      |
  +--------+--------+--------+

  One valid sequence of operations:
    horse -> rorse  (replace h with r)
    rorse -> rose   (delete r at position 2)
    rose  -> ros    (delete e)
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

int editDistance(const string& word1, const string& word2) {
    int m = word1.size(), n = word2.size();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

    // Base cases
    for (int i = 0; i <= m; i++) dp[i][0] = i;  // delete all
    for (int j = 0; j <= n; j++) dp[0][j] = j;  // insert all

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1[i-1] == word2[j-1]) {
                dp[i][j] = dp[i-1][j-1];  // match, no cost
            } else {
                dp[i][j] = 1 + min({
                    dp[i-1][j],      // delete
                    dp[i][j-1],      // insert
                    dp[i-1][j-1]     // replace
                });
            }
        }
    }
    return dp[m][n];
}

int main() {
    cout << "Edit distance (horse->ros): "
         << editDistance("horse", "ros") << endl;  // 3
    return 0;
}
```

### Related: Longest Palindromic Subsequence

The longest palindromic subsequence of string s equals **LCS(s, reverse(s))**.
No new algorithm needed -- just reuse LCS.

```
  Example: s = "bbbab"
  reverse(s) = "babbb"
  LCS("bbbab", "babbb") = "bbbb" (length 4)
  So the longest palindromic subsequence has length 4.
```

---

## 🔹 Pattern 7: Interval DP

**Problem**: Matrix Chain Multiplication -- given matrices A1 x A2 x ... x An
with dimensions p[0] x p[1], p[1] x p[2], ..., p[n-1] x p[n], find the
parenthesization that minimizes total scalar multiplication cost.

**State**: `dp[i][j]` = minimum cost to multiply matrices i through j

**Recurrence**:
```
dp[i][j] = min over all k in [i, j-1]:
    dp[i][k] + dp[k+1][j] + p[i-1] * p[k] * p[j]

    |           |           |
    cost of     cost of     cost of multiplying
    left half   right half  the two result matrices
```

**Base case**: `dp[i][i] = 0` (single matrix costs nothing)

**Iteration order**: by increasing chain length L = j - i + 1

### Worked Example

```
  Matrices with dimensions [10, 30, 5, 60]
    A1 is 10x30,  A2 is 30x5,  A3 is 5x60

  Possible parenthesizations:
    (A1 * A2) * A3:  10*30*5 + 10*5*60  = 1500 + 3000  = 4500
    A1 * (A2 * A3):  30*5*60 + 10*30*60 = 9000 + 18000 = 27000

  dp table (fill by chain length):

            j=1     j=2     j=3
    i=1  [   0    1500    4500 ]
    i=2  [   -       0    9000 ]
    i=3  [   -       -       0 ]

  Fill order:
    L=1 (single matrices):
      dp[1][1] = 0
      dp[2][2] = 0
      dp[3][3] = 0

    L=2 (pairs):
      dp[1][2] = p[0]*p[1]*p[2] = 10*30*5 = 1500  (only k=1)
      dp[2][3] = p[1]*p[2]*p[3] = 30*5*60 = 9000  (only k=2)

    L=3 (full chain):
      dp[1][3] = min(
        k=1: dp[1][1] + dp[2][3] + p[0]*p[1]*p[3]
             = 0 + 9000 + 10*30*60 = 27000
        k=2: dp[1][2] + dp[3][3] + p[0]*p[2]*p[3]
             = 1500 + 0 + 10*5*60 = 4500           <-- BEST
      ) = 4500

  Answer: 4500, parenthesized as (A1 * A2) * A3
```

**Where this pattern appears**:
- **Burst Balloons** (LeetCode 312) -- pop balloons in optimal order
- **Minimum Cost to Merge Stones** -- merge adjacent piles
- **Palindrome Partitioning II** -- minimum cuts
- **Optimal Binary Search Tree** -- minimize expected search cost
- Any problem where you **split a range** and combine results from both halves

```
Time:  O(n^3) -- three nested loops (length, i, k)
Space: O(n^2)
```

---

## 🔹 Summary Table

| Pattern | State | Time | Space | Classic Problem |
|---|---|---|---|---|
| 0/1 Knapsack | `dp[i][w]` | O(nW) | O(W) optimized | Subset Sum, Equal Partition |
| Unbounded Knapsack | `dp[amount]` | O(n * amount) | O(amount) | Coin Change |
| LCS | `dp[i][j]` | O(mn) | O(mn) | Diff tool, DNA alignment |
| LIS | `dp[i]` | O(n^2) or O(n log n) | O(n) | Patience Sorting |
| Grid Paths | `dp[i][j]` | O(mn) | O(n) optimized | Unique Paths, Min Path Sum |
| String DP | `dp[i][j]` | O(mn) | O(mn) | Edit Distance, Spell Check |
| Interval DP | `dp[i][j]` | O(n^3) | O(n^2) | Matrix Chain, Burst Balloons |

### Pattern Recognition Flowchart

```
  What does the problem look like?
  |
  +-- Selecting items with constraints?
  |   +-- Each item used at most once?     --> 0/1 Knapsack
  |   +-- Items can be reused?             --> Unbounded Knapsack
  |
  +-- Comparing two sequences?
  |   +-- Find common structure?           --> LCS
  |   +-- Transform one into the other?    --> Edit Distance (String DP)
  |
  +-- Working with one sequence?
  |   +-- Find longest ordered subseq?     --> LIS
  |
  +-- Moving through a grid?               --> Grid DP
  |
  +-- Splitting/merging a range?
      +-- Try all split points?            --> Interval DP
```

---

**Key insight**: DP is not about memorizing solutions. It is about recognizing
the *structure* of the subproblems. Once you see which pattern fits, the
recurrence writes itself.

See also: [[Memoization vs Tabulation]] | [[Recursion]] | [[Backtracking]] | [[Greedy Technique]]
