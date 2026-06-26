---
tags:
  - algorithms
  - pattern-recognition
  - greedy
---

## 🔹 When to Suspect This Pattern

- **Optimization** problem — "minimum number of X", "maximum number of Y"
- **Interval / scheduling** problems — "meeting rooms", "activity selection", "merge intervals"
- Making a **locally optimal choice** at each step seems sufficient
- Keywords: "minimum coins (specific denominations)", "jump game", "assign tasks", "merge"
- The problem has the **greedy choice property**: a locally optimal choice leads to a globally optimal solution
- Simpler than DP — if greedy works, it's preferred (lower complexity)

---

## 🔹 Confirming It's the Right Pattern

The **Exchange Argument**: Can you prove that swapping any non-greedy choice for the greedy choice never makes the solution worse?

Checklist:
- [ ] Does making the locally best choice always lead to a valid global solution?
- [ ] Can you prove (even informally) that no greedy choice needs to be "undone"?
- [ ] Is there a natural **sorting order** that reveals the greedy strategy?
- [ ] If you can't convince yourself greedy works, fall back to [[Pattern - Dynamic Programming]]

> [!warning] Greedy Traps
> These problems LOOK greedy but aren't:
> - **Coin Change (arbitrary denominations)**: greedy fails for `[1, 3, 4]` with target 6 (greedy: 4+1+1=3, optimal: 3+3=2)
> - **0/1 Knapsack**: greedy by value/weight ratio doesn't always work
> - **Longest Increasing Subsequence**: no greedy solution
>
> If in doubt, test your greedy on small counterexamples before committing.

---

## 🔹 Common Greedy Strategies

### Strategy 1: Sort and Process

Most greedy problems start with sorting by some property.

| Sort By | Problem Type |
|---|---|
| End time (ascending) | Activity/interval selection (maximize count) |
| Start time (ascending) | Merge intervals, meeting rooms |
| Deadline (ascending) | Task scheduling |
| Size/weight ratio | Fractional knapsack |
| Value (ascending/descending) | Assigning tasks optimally |

### Strategy 2: Always Pick the Extreme

Take the smallest/largest available element that satisfies the constraint.

### Strategy 3: Process Greedily with a Heap

Use a min/max heap to always have access to the "best" next choice.

```cpp
// Meeting Rooms II: minimum rooms needed
sort(intervals.begin(), intervals.end());  // sort by start time
priority_queue<int, vector<int>, greater<int>> pq;  // min-heap of end times

for (auto& interval : intervals) {
    if (!pq.empty() && pq.top() <= interval.start)
        pq.pop();  // reuse room
    pq.push(interval.end);
}
return pq.size();
```

---

## 🔹 Templates

### Interval Scheduling (Maximum Non-Overlapping)

```cpp
// Sort by END time — key insight for maximizing count
sort(intervals.begin(), intervals.end(),
     [](auto& a, auto& b) { return a.end < b.end; });

int count = 0, lastEnd = INT_MIN;
for (auto& interval : intervals) {
    if (interval.start >= lastEnd) {
        count++;
        lastEnd = interval.end;
    }
}
```

### Merge Intervals

```cpp
sort(intervals.begin(), intervals.end());  // sort by start
vector<vector<int>> merged;

for (auto& interval : intervals) {
    if (merged.empty() || merged.back()[1] < interval[0])
        merged.push_back(interval);       // no overlap
    else
        merged.back()[1] = max(merged.back()[1], interval[1]);  // extend
}
```

### Jump Game (Can You Reach End?)

```cpp
int farthest = 0;
for (int i = 0; i < n; i++) {
    if (i > farthest) return false;  // stuck
    farthest = max(farthest, i + arr[i]);
}
return true;
```

---

## 🔹 Classic Problems

| Problem | Greedy Strategy | Why Greedy Works |
|---|---|---|
| **Activity Selection** | Sort by end time, pick earliest finishing | Leaves maximum room for future activities |
| **Merge Intervals** | Sort by start, extend overlaps | Processing in order guarantees no overlap missed |
| **Jump Game** | Track farthest reachable index | If you can reach i, you can reach anything before i |
| **Jump Game II (min jumps)** | BFS-style: track current range | Each "level" is one jump |
| **Assign Cookies** | Sort both, match smallest cookie to smallest child | Wastes nothing |
| **Gas Station** | Track running surplus, reset at deficit | If total gas >= total cost, solution exists |
| **Task Scheduler** | Fill most frequent tasks first, idle between | Minimizes total idle time |
| **Non-overlapping Intervals** | Sort by end, count overlaps to remove | Same as activity selection (inverted) |

---

## 🔹 Greedy vs DP Quick Check

| Signal | Greedy | DP |
|---|---|---|
| Can prove exchange argument | **Use Greedy** | — |
| Optimal substructure + no overlap | **Use Greedy** | — |
| Overlapping subproblems | — | **Use DP** |
| Counterexample found for greedy | — | **Use DP** |
| Problem says "all ways" or "count" | — | **Use DP** (greedy finds one, not all) |
| Interval scheduling | Usually **Greedy** | — |
| Knapsack (0/1) | — | **Use DP** |
| Coin Change (standard denominations) | **Greedy** works | — |
| Coin Change (arbitrary denominations) | — | **Use DP** |

---

## 🔹 Common Mistakes

- **Not proving greedy works**: always at least test with 2-3 small examples and look for counterexamples
- **Wrong sorting criterion**: "sort by start" vs "sort by end" can make or break the solution. Activity selection = sort by END
- **Not handling ties**: when values are equal, the tiebreaker matters (e.g., sort by end, then by start)
- **Confusing "can reach" (greedy) with "min steps" (BFS-greedy hybrid)**: Jump Game I is pure greedy; Jump Game II needs BFS-like level tracking
- **Applying greedy to DP problems**: if unsure, always try to find a counterexample before committing to greedy

---

## 🔹 Related Patterns

- [[Pattern - Dynamic Programming]] — fallback when greedy doesn't work
- [[Pattern - Binary Search]] — "binary search on answer" often uses greedy to validate
- [[Pattern - Graph]] — MST (Kruskal's, Prim's) are greedy graph algorithms
- [[How to Pick the Right Algorithm]] — greedy vs DP decision guide
