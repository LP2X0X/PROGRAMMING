---
tags:
  - algorithms
  - technique
  - backtracking
---
# Backtracking

## 🔹 Real-World Analogy

**Solving a maze:** You walk forward at every intersection, picking a direction. When you hit a dead end, you **backtrack** to the last intersection and try a different path. You keep doing this until you find the exit — or exhaust all paths.

**Trying on outfits:** You pick a shirt, then pants, then shoes. If the outfit doesn't look right, you put the shoes back, try different shoes. Still bad? Put the pants back too, try different pants. You systematically try combinations by making a choice, evaluating, and undoing.

```
MAZE ANALOGY:

    S → → → ╗         S = Start
    ↑       ↓         E = End
    ↑   ╔ ← ╝         ╗╔╝╚ = walls/dead ends
    ↑   ↓
    ↑   ╚ → → E

    You try a path. Dead end? Go back.
    Try the next path. Dead end? Go back.
    Eventually you find S → → → → ↓ → → E
```

---

## 🔹 What is Backtracking?

**Systematic trial-and-error.** An algorithmic technique where you:

1. **Make a choice** from available options
2. **Explore** what happens by recursing deeper
3. If it leads to a **dead end**, **UNDO the choice** and try the next option

It builds a **decision tree** and explores it using [[DFS - Depth First Search]], **abandoning branches** that can't lead to a valid solution.

```
BACKTRACKING = Recursion + Choose + Undo

    ┌─────────┐     ┌──────────┐     ┌──────────┐
    │  CHOOSE │ ──→ │  EXPLORE │ ──→ │ UNCHOOSE │
    │ (pick)  │     │ (recurse)│     │  (undo)  │
    └─────────┘     └──────────┘     └──────────┘
         ↑                                │
         └────────────────────────────────┘
              try next option
```

**How is it different from plain [[Recursion]]?**

| Plain Recursion | Backtracking |
|----------------|-------------|
| Breaks problem into subproblems | Builds solutions by making choices |
| No "undoing" | Explicitly undoes choices |
| One path through | Explores multiple paths |
| Computes a result | Searches for valid configurations |

---

## 🔹 Decision Tree Visualization

Every backtracking problem can be visualized as a **decision tree**. Each node is a state, each edge is a choice.

### Generating all subsets of {1, 2, 3}:

At each level, decide: **include this element or skip it?**

```
                              []
                       /              \
                  include 1          skip 1
                    [1]                []
                 /       \          /       \
            incl 2     skip 2   incl 2    skip 2
            [1,2]       [1]      [2]        []
            /   \      /   \    /   \      /   \
          i3    s3   i3    s3  i3   s3   i3    s3
       [1,2,3][1,2][1,3] [1] [2,3] [2]  [3]   []
         ▲     ▲     ▲    ▲    ▲    ▲    ▲     ▲
         └─────┴─────┴────┴────┴────┴────┴─────┘
                    All 8 subsets (2^3)
```

### Generating permutations of {1, 2, 3}:

At each level, choose: **which remaining element goes next?**

```
                              []
                    /          |          \
                pick 1       pick 2      pick 3
                 [1]          [2]          [3]
               /    \       /    \       /    \
             p2     p3    p1     p3    p1     p2
            [1,2]  [1,3] [2,1]  [2,3] [3,1]  [3,2]
              |      |     |      |     |       |
             p3     p2    p3     p1    p2      p1
           [1,2,3][1,3,2][2,1,3][2,3,1][3,1,2][3,2,1]
             ▲      ▲      ▲      ▲      ▲       ▲
             └──────┴──────┴──────┴──────┴───────┘
                       All 6 permutations (3!)
```

---

## 🔹 The Choose-Explore-Unchoose Pattern — THE CORE

This is the heart of every backtracking solution:

```cpp
void backtrack(State& state, Choices& choices) {
    // Goal: found a complete solution?
    if (is_solution(state)) {
        record(state);      // save it, print it, count it
        return;
    }
    
    for (auto& choice : choices) {
        if (is_valid(choice, state)) {
            make_choice(state, choice);      // ← CHOOSE
            backtrack(state, next_choices);   // ← EXPLORE
            undo_choice(state, choice);       // ← UNCHOOSE  *** THIS IS THE KEY ***
        }
    }
}
```

**Why UNCHOOSE?** Because after exploring one branch, you need to restore the state so the next branch starts from the same clean state. Without undoing, each branch would see the mess left by the previous one.

```
Without UNCHOOSE:              With UNCHOOSE:

State: [1]                     State: [1]
  explore → [1,2]                explore → [1,2]
  explore → [1,2,3]  ✓            undo → [1]
  next?  → [1,2,3,?] ✗          explore → [1,3]
  BROKEN!                          undo → [1]
                                 CLEAN! Each branch sees [1]
```

---

## 🔹 When to Use Backtracking

Use backtracking when the problem asks you to:

| Problem Pattern | Example |
|-----------------|---------|
| Find **ALL** permutations | Arrange letters/numbers in every order |
| Find **ALL** combinations/subsets | Choose k items from n |
| **Constraint satisfaction** | N-Queens, Sudoku, crossword puzzles |
| **Path finding** | All paths in a maze, word search in a grid |
| Generate **ALL valid** configurations | Valid parentheses, IP addresses |
| Find **ANY** solution satisfying constraints | Sudoku solver, map coloring |

**Key signal words in problem statements:**
- "Find all..."
- "Generate all..."
- "List every..."
- "Is there a way to..."
- "How many ways..."
- "Print all valid..."

---

## 🔹 Pruning — Making Backtracking Practical

Without pruning, backtracking explores **every** branch of the decision tree. Pruning cuts off branches early when they **cannot possibly** lead to a valid solution.

```
WITHOUT PRUNING (brute force):       WITH PRUNING:

         [root]                           [root]
        / | | \                          / |   \
       A  B  C  D                       A  B    D
      /|\ ...                          /|       |
     ...                              E F       G
                                        |
    Explores ALL branches               H
    Even obviously bad ones
                                    Skips C entirely (invalid)
                                    Skips subtrees of A that fail
```

### Example: N-Queens Pruning

When placing queens row by row, if placing a queen on row 2 creates a diagonal conflict with row 1, **don't explore any placements for rows 3, 4, ..., n** in that branch. The conflict won't magically resolve itself.

```
Pruning in 4-Queens:

Row 0:  Q . . .     Place queen at (0,0)
Row 1:  . . Q .     Place queen at (1,2) — no conflict
Row 2:  Can't place anywhere without conflict!
        
        → PRUNE! Don't try row 3.
        → Backtrack to row 1, try (1,3) instead.

Savings: Avoided exploring all row 2 × row 3 combinations
         for this dead-end branch.
```

**Impact of pruning:**

| Problem | Without Pruning | With Smart Pruning |
|---------|----------------|-------------------|
| 8-Queens | 16,777,216 nodes | ~15,000 nodes |
| Sudoku | 9^81 possibilities | Typically < 10,000 |

---

## 🔹 Time Complexity

Backtracking is inherently **exponential**, but pruning can cut the practical work dramatically.

| Problem Type | Typical Complexity |
|-------------|-------------------|
| Subsets | O(2^n) |
| Permutations | O(n!) |
| Combinations C(n,k) | O(C(n,k)) |
| N-Queens | O(n!) worst case |
| Sudoku | O(9^(empty cells)) worst case |

**Space complexity:** O(depth of decision tree) for the call stack, plus space for storing solutions.

---

## 🔹 Template Code with Full Implementations

### Implementation 1: Generate All Permutations of [1, 2, 3]

```cpp
#include <vector>
using namespace std;

void permute(vector<int>& nums, int start, vector<vector<int>>& result) {
    // Base case: all positions filled
    if (start == nums.size()) {
        result.push_back(nums);
        return;
    }
    
    for (int i = start; i < nums.size(); i++) {
        swap(nums[start], nums[i]);          // CHOOSE: place nums[i] at position start
        permute(nums, start + 1, result);    // EXPLORE: fill remaining positions
        swap(nums[start], nums[i]);          // UNCHOOSE: restore original order
    }
}
```

**Step-by-step trace for [1, 2, 3]:**

```
permute([1,2,3], start=0)
│
├─ swap(0,0): [1,2,3]  → permute([1,2,3], start=1)
│  ├─ swap(1,1): [1,2,3] → permute([1,2,3], start=2) → RECORD [1,2,3] ✓
│  │  └─ swap(1,1): [1,2,3]  (undo)
│  ├─ swap(1,2): [1,3,2] → permute([1,3,2], start=2) → RECORD [1,3,2] ✓
│  │  └─ swap(1,2): [1,2,3]  (undo)
│  └─ swap(0,0): [1,2,3]  (undo)
│
├─ swap(0,1): [2,1,3]  → permute([2,1,3], start=1)
│  ├─ swap(1,1): [2,1,3] → permute([2,1,3], start=2) → RECORD [2,1,3] ✓
│  ├─ swap(1,2): [2,3,1] → permute([2,3,1], start=2) → RECORD [2,3,1] ✓
│  └─ swap(0,1): [1,2,3]  (undo)
│
├─ swap(0,2): [3,2,1]  → permute([3,2,1], start=1)
│  ├─ swap(1,1): [3,2,1] → permute([3,2,1], start=2) → RECORD [3,2,1] ✓
│  ├─ swap(1,2): [3,1,2] → permute([3,1,2], start=2) → RECORD [3,1,2] ✓
│  └─ swap(0,2): [1,2,3]  (undo)

Result: [1,2,3] [1,3,2] [2,1,3] [2,3,1] [3,2,1] [3,1,2]  → 3! = 6 permutations
```

---

### Implementation 2: Generate All Subsets of [1, 2, 3]

```cpp
void subsets(vector<int>& nums, int index, vector<int>& current,
             vector<vector<int>>& result) {
    // Base case: considered all elements
    if (index == nums.size()) {
        result.push_back(current);
        return;
    }
    
    // Choice 1: INCLUDE nums[index]
    current.push_back(nums[index]);          // CHOOSE
    subsets(nums, index + 1, current, result); // EXPLORE
    current.pop_back();                       // UNCHOOSE
    
    // Choice 2: EXCLUDE nums[index]
    subsets(nums, index + 1, current, result); // EXPLORE (no choose/unchoose needed)
}
```

**Step-by-step trace for [1, 2, 3]:**

```
subsets([1,2,3], idx=0, current=[])
│
├─ INCLUDE 1: current=[1]
│  ├─ INCLUDE 2: current=[1,2]
│  │  ├─ INCLUDE 3: current=[1,2,3] → RECORD [1,2,3] ✓
│  │  └─ EXCLUDE 3: current=[1,2]   → RECORD [1,2]   ✓
│  └─ EXCLUDE 2: current=[1]
│     ├─ INCLUDE 3: current=[1,3]   → RECORD [1,3]   ✓
│     └─ EXCLUDE 3: current=[1]     → RECORD [1]     ✓
│
└─ EXCLUDE 1: current=[]
   ├─ INCLUDE 2: current=[2]
   │  ├─ INCLUDE 3: current=[2,3]   → RECORD [2,3]   ✓
   │  └─ EXCLUDE 3: current=[2]     → RECORD [2]     ✓
   └─ EXCLUDE 2: current=[]
      ├─ INCLUDE 3: current=[3]     → RECORD [3]     ✓
      └─ EXCLUDE 3: current=[]      → RECORD []      ✓

Result: [1,2,3] [1,2] [1,3] [1] [2,3] [2] [3] []  → 2^3 = 8 subsets
```

---

### Implementation 3: N-Queens Problem

Place N queens on an NxN chessboard so no two queens attack each other (same row, column, or diagonal).

**Solution for N=4:**

```
Board 1:            Board 2:
. Q . .             . . Q .
. . . Q             Q . . .
Q . . .             . . . Q
. . Q .             . Q . .

Q = Queen            Two solutions exist for N=4
. = Empty
```

**Attack patterns — what to check:**

```
      col                 diag1 (row-col)       diag2 (row+col)
       ↓                      ╲                       ╱
  . . Q . .              . . . . ╲              ╱ . . . .
  . . ↓ . .              . . . ╲ .              . ╱ . . .
  . . ↓ . .              . . ╲ . .              . . ╱ . .
  . . ↓ . .              . ╲ . . .              . . . ╱ .
  . . ↓ . .              ╲ . . . .              . . . . ╱

  Same column:            row - col = constant   row + col = constant
  col is taken            diagonal ╲ is taken    diagonal ╱ is taken
```

**Full implementation:**

```cpp
#include <vector>
#include <string>
using namespace std;

class NQueens {
    int n;
    vector<bool> cols;       // columns with queens
    vector<bool> diag1;      // row - col + (n-1) diagonals  (╲)
    vector<bool> diag2;      // row + col diagonals          (╱)
    vector<vector<string>> results;
    
public:
    vector<vector<string>> solve(int n) {
        this->n = n;
        cols.assign(n, false);
        diag1.assign(2 * n - 1, false);
        diag2.assign(2 * n - 1, false);
        
        vector<string> board(n, string(n, '.'));
        backtrack(board, 0);
        return results;
    }
    
private:
    void backtrack(vector<string>& board, int row) {
        // Base case: all queens placed
        if (row == n) {
            results.push_back(board);
            return;
        }
        
        // Try placing a queen in each column of this row
        for (int col = 0; col < n; col++) {
            int d1 = row - col + (n - 1);   // ╲ diagonal index
            int d2 = row + col;              // ╱ diagonal index
            
            // PRUNE: skip if any conflict
            if (cols[col] || diag1[d1] || diag2[d2]) {
                continue;
            }
            
            // CHOOSE
            board[row][col] = 'Q';
            cols[col] = diag1[d1] = diag2[d2] = true;
            
            // EXPLORE
            backtrack(board, row + 1);
            
            // UNCHOOSE
            board[row][col] = '.';
            cols[col] = diag1[d1] = diag2[d2] = false;
        }
    }
};
```

**Trace for N=4 (first solution found):**

```
Row 0: Try col 0
  . . . .       Q placed at (0,0)
  Q → . . .

Row 1: col 0 conflict (column), col 1 conflict (diagonal)
       Try col 2
  Q . . .
  . . Q .       Q placed at (1,2)

Row 2: col 0,1,2,3 — only col 1 works? No.
       col 0: column conflict with row 0
       col 1: diagonal conflict with row 1
       col 2: column conflict with row 1
       col 3: diagonal conflict with row 1
       ALL BLOCKED → BACKTRACK to row 1

Row 1: Undo (1,2). Try col 3
  Q . . .
  . . . Q       Q placed at (1,3)

Row 2: Try col 1
  Q . . .
  . . . Q
  . Q . .       Q placed at (2,1)

Row 3: col 0: diag conflict, col 1: col conflict,
       col 2: no conflict!
  Q . . .
  . . . Q
  . Q . .
  . . Q .       Q placed at (3,2) → SOLUTION FOUND! ✓
```

---

## 🔹 Backtracking vs Other Approaches

| When the problem has... | Use | Why |
|------------------------|-----|-----|
| Overlapping subproblems + optimal substructure | [[Dynamic Programming]] | Avoids recomputation via caching |
| Greedy choice property | [[Greedy Technique]] | Local optimal → global optimal |
| Need ALL or ANY valid configurations | **Backtracking** | Systematic exploration of choices |
| Simple optimal substructure only | [[Recursion]] | Direct divide and conquer |
| Shortest path (unweighted) | [[BFS - Breadth First Search]] | Level-by-level guarantees shortest |
| Exploring all reachable states | [[DFS - Depth First Search]] | Backtracking IS DFS on a decision tree |

**Key distinction from [[Dynamic Programming]]:**
- DP: "What is the **optimal value**?" → overlapping subproblems, cache results
- Backtracking: "What are **all valid configurations**?" → explore and undo

Sometimes they combine: if backtracking has overlapping states, add [[Memoization vs Tabulation]] to get DP.

---

## 🔹 Common Pitfalls

### 1. Forgetting to Unchoose
```cpp
// WRONG — state accumulates garbage from previous branches
current.push_back(choice);
backtrack(state);
// missing: current.pop_back();
```

### 2. Modifying the Loop Variable
```cpp
// WRONG — skips elements or infinite loop
for (int i = start; i < n; i++) {
    i++;  // accidentally modifying loop counter
    backtrack(i);
}
```

### 3. Not Pruning
Without pruning, backtracking degenerates into brute force. Always ask: "Can I detect failure early and skip this branch?"

### 4. Confusing Permutations vs Combinations
- **Permutations:** order matters, [1,2] != [2,1], use swap or visited array
- **Combinations:** order doesn't matter, [1,2] == [2,1], use start index to avoid duplicates

### 5. Duplicate Results with Repeated Elements
For input [1, 2, 2], naive backtracking generates duplicate subsets. Fix by sorting the input and skipping consecutive duplicates:

```cpp
// Skip duplicates in combination/subset generation
if (i > start && nums[i] == nums[i-1]) continue;
```

---

## 🔹 See Also

- [[Recursion]] — the foundation that backtracking builds on
- [[Dynamic Programming]] — when backtracking subproblems overlap, cache them
- [[DFS - Depth First Search]] — backtracking is DFS on a decision tree
- [[Memoization vs Tabulation]] — top-down caching to speed up recursive search
- [[Common DP Patterns]] — recognize when backtracking can be optimized to DP
