---
tags:
  - algorithms
  - pattern-recognition
  - backtracking
---

## 🔹 When to Suspect This Pattern

- Keywords: "**generate all**", "**find all valid**", "**all permutations**", "**all combinations**", "**all subsets**"
- **Constraint satisfaction** problems — Sudoku, N-Queens, crossword
- Need to **explore all possibilities** and collect valid ones
- The solution is built **incrementally**, choosing one element at a time
- Dead ends exist — you need to **undo choices** and try alternatives
- Input size is **small** (n <= 15-20) — see [[Constraint Analysis]]

---

## 🔹 Confirming It's the Right Pattern

- [ ] Does the problem ask to generate or enumerate **all valid** solutions?
- [ ] Is the input small enough for exponential time? (n <= ~20 for 2^n, n <= ~10 for n!)
- [ ] Can you build the solution **one choice at a time**?
- [ ] Are there **constraints** that let you prune invalid branches early?
- [ ] Is the solution a **sequence of decisions** (pick or skip, place or don't)?

> [!tip] Backtracking = DFS on the Decision Tree
> Think of backtracking as DFS on an implicit tree where:
> - Each **level** = one decision
> - Each **branch** = one choice
> - **Leaves** = complete solutions
> - **Pruning** = skipping subtrees that can't lead to valid solutions

---

## 🔹 The Universal Template

```cpp
void backtrack(vector<...>& result, vector<...>& current,
               /* remaining choices / state */) {
    // BASE CASE: solution is complete
    if (is_complete(current)) {
        result.push_back(current);
        return;
    }

    for (each choice in available_choices) {
        // PRUNE: skip invalid choices early
        if (!is_valid(choice)) continue;

        // CHOOSE: make the choice
        current.push_back(choice);
        // or: mark as used, place on board, etc.

        // EXPLORE: recurse with updated state
        backtrack(result, current, next_state);

        // UNCHOOSE: undo the choice (backtrack)
        current.pop_back();
        // or: unmark, remove from board, etc.
    }
}
```

---

## 🔹 Three Core Variants

### Variant 1: Subsets (Include or Exclude)

Each element is either included or not. No ordering matters.

```cpp
void subsets(vector<int>& nums, int start,
             vector<int>& current, vector<vector<int>>& result) {
    result.push_back(current);  // every state is a valid subset

    for (int i = start; i < nums.size(); i++) {
        current.push_back(nums[i]);
        subsets(nums, i + 1, current, result);  // i+1: no reuse
        current.pop_back();
    }
}
```

### Variant 2: Permutations (All Orderings)

Every element is used exactly once, order matters.

```cpp
void permute(vector<int>& nums, vector<bool>& used,
             vector<int>& current, vector<vector<int>>& result) {
    if (current.size() == nums.size()) {
        result.push_back(current);
        return;
    }

    for (int i = 0; i < nums.size(); i++) {
        if (used[i]) continue;
        used[i] = true;
        current.push_back(nums[i]);
        permute(nums, used, current, result);
        current.pop_back();
        used[i] = false;
    }
}
```

### Variant 3: Combinations (Choose K from N)

Select exactly K elements, order doesn't matter.

```cpp
void combine(int n, int k, int start,
             vector<int>& current, vector<vector<int>>& result) {
    if (current.size() == k) {
        result.push_back(current);
        return;
    }

    for (int i = start; i <= n; i++) {
        current.push_back(i);
        combine(n, k, i + 1, current, result);
        current.pop_back();
    }
}
```

---

## 🔹 Handling Duplicates

When input contains duplicates, sort first and skip consecutive identical elements:

```cpp
sort(nums.begin(), nums.end());
// Inside the loop:
for (int i = start; i < nums.size(); i++) {
    if (i > start && nums[i] == nums[i - 1]) continue;  // skip duplicates
    // ... choose, explore, unchoose
}
```

---

## 🔹 Classic Problems

| Problem | Type | Key Pruning |
|---|---|---|
| **Subsets** | Subset | None needed (all valid) |
| **Subsets II** (with duplicates) | Subset | Sort + skip consecutive duplicates |
| **Permutations** | Permutation | `used[]` array |
| **Permutations II** (with duplicates) | Permutation | Sort + skip same-level duplicates |
| **Combination Sum** | Combination (with reuse) | Start from current index (allow reuse) |
| **Combination Sum II** (no reuse) | Combination | Start from `i+1` + skip duplicates |
| **N-Queens** | Constraint satisfaction | Column, diagonal, anti-diagonal checks |
| **Sudoku Solver** | Constraint satisfaction | Row, column, box constraints |
| **Word Search** | Grid DFS | Bounds + visited check |
| **Palindrome Partitioning** | Partition | Check palindrome before recursing |
| **Letter Combinations of Phone** | Combination | Map digit → letters |
| **Generate Parentheses** | Custom | Open < n, close < open |

---

## 🔹 Pruning Strategies

| Strategy | How It Works | Impact |
|---|---|---|
| **Constraint check** | Skip choices that violate constraints | Eliminates invalid branches |
| **Sort + skip duplicates** | After sorting, skip `nums[i] == nums[i-1]` | Avoids duplicate solutions |
| **Remaining elements check** | If remaining elements can't complete solution, stop | Early termination |
| **Upper/lower bound** | If best possible from here can't beat current best, stop | Branch-and-bound |
| **Symmetry breaking** | Fix first choice to avoid symmetric solutions | Cuts search space in half (or more) |

---

## 🔹 Common Mistakes

- **Forgetting to undo the choice**: every `push_back` needs a `pop_back`, every `used[i] = true` needs `used[i] = false`
- **Wrong starting index**: subsets/combinations use `i + 1` (no reuse); allowing reuse uses `i` (same element again)
- **Duplicate handling**: must **sort first**, then skip `nums[i] == nums[i-1]` only when `i > start` (not `i > 0`)
- **Copying vs referencing**: `result.push_back(current)` copies the vector. If using references elsewhere, ensure you're pushing a copy, not a reference that gets modified
- **Not pruning enough**: without pruning, backtracking is just brute force. Always ask "can I reject this branch early?"

---

## 🔹 Backtracking vs Other Patterns

| If the problem... | Use | Why |
|---|---|---|
| Says "all" / "generate" / "find every" | **Backtracking** | Need to enumerate |
| Says "count the number of ways" | [[Pattern - Dynamic Programming]] | DP counts without generating |
| Says "find the optimal" | [[Pattern - Dynamic Programming]] or [[Pattern - Greedy]] | Don't need all solutions |
| Has overlapping subproblems | **Backtracking + Memoization = DP** | Add memo to avoid recomputation |

---

## 🔹 Related Patterns

- [[Pattern - Dynamic Programming]] — DP = backtracking + memoization (eliminates redundant work)
- [[Pattern - Tree]] — backtracking is DFS on an implicit decision tree
- [[Pattern - Graph]] — grid search problems use backtracking-style DFS
- [[Constraint Analysis]] — backtracking only viable for small n (n <= 15-20)
