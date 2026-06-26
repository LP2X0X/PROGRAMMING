---
tags:
  - algorithms
  - technique
  - greedy
related:
  - "[[Dynamic Programming]]"
  - "[[Memoization vs Tabulation]]"
  - "[[Common DP Patterns]]"
  - "[[Backtracking]]"
  - "[[Recursion]]"
  - "[[Priority Queue]]"
---

## 🔹 Real-World Analogy: Making Change

Suppose someone owes you **$0.67**. Without thinking, you grab:

```
Amount remaining: $0.67

Step 1: Grab a quarter (25c)  → remaining: $0.42
Step 2: Grab a quarter (25c)  → remaining: $0.17
Step 3: Grab a dime   (10c)   → remaining: $0.07
Step 4: Grab a nickel  (5c)   → remaining: $0.02
Step 5: Grab a penny   (1c)   → remaining: $0.01
Step 6: Grab a penny   (1c)   → remaining: $0.00

Total coins: 6  ← optimal!

  $0.67 = 25 + 25 + 10 + 5 + 1 + 1
           ↑    ↑    ↑   ↑   ↑   ↑
        biggest first ... smallest last
```

You never considered all possible combinations. You just grabbed the **biggest coin that fits** each time. That is the greedy approach — and it happens to produce the optimal answer for US coins.

**But it does NOT always work!** With coins `{1, 3, 4}` and amount `6`:

```
GREEDY:   4 + 1 + 1 = 3 coins   ← WRONG (not optimal)
OPTIMAL:  3 + 3     = 2 coins   ← needs DP!

Why? Picking 4 first "locks us in" — we can't undo it,
and the remaining 2 can only be made with 1+1.
The better path (3+3) required NOT picking the largest coin.
```

This is the core tension of greedy: **fast and simple, but only correct under specific conditions.**

---

## 🔹 What Is the Greedy Technique?

At each step, make the choice that looks **best right now** — and never look back.

```
 Problem: ─────────────────────────────────────────────►

 GREEDY builds solution step by step:

   Step 1          Step 2          Step 3
   ┌─────┐        ┌─────┐        ┌─────┐
   │Pick  │───────│Pick  │───────│Pick  │───── ... ──► Solution
   │BEST  │       │BEST  │       │BEST  │
   │now   │       │now   │       │now   │
   └─────┘        └─────┘        └─────┘
      │               │               │
      ▼               ▼               ▼
   COMMITTED      COMMITTED      COMMITTED
   (no undo)      (no undo)      (no undo)
```

**Key characteristics:**
- **No backtracking** — once a choice is made, it is final
- **No exhaustive search** — only one option considered per step
- **Hope** that local optimality leads to global optimality
- **Much simpler and faster** than [[Dynamic Programming]]

Greedy only works when **BOTH** conditions hold:

### a) Greedy Choice Property

A **locally optimal** choice is part of some **globally optimal** solution. You can safely commit to the best-looking option right now without worrying about future consequences.

### b) Optimal Substructure

The optimal solution to the whole problem **contains** optimal solutions to its subproblems. (This is shared with DP — see [[Dynamic Programming]].)

```
┌───────────────────────────────────────────────────────────┐
│  GREEDY = Greedy Choice Property + Optimal Substructure   │
│                                                           │
│  If EITHER property fails → greedy gives WRONG answers    │
│  (not just suboptimal — genuinely incorrect!)             │
└───────────────────────────────────────────────────────────┘
```

---

## 🔹 Greedy vs Dynamic Programming — The Key Difference

```
 GREEDY:
 ┌──────────────┐     ┌───────────────────────────┐
 │ Make choice   │────►│ Solve remaining subproblem │────► Answer
 │ (locally best)│     │ (only ONE path forward)    │
 └──────────────┘     └───────────────────────────┘
   Never look back. Never reconsider.


 DP:
 ┌──────────────┐     ┌───────────────────────────┐
 │ Try ALL       │────►│ Solve ALL subproblems      │
 │ choices       │     │ (explore every path)       │
 └──────────────┘     └───────────────────────────┘
                                │
                                ▼
                       Pick the BEST across all


                GREEDY                              DP
           ┌──────────┐                     ┌──────────┐
           │ Problem  │                     │ Problem  │
           └────┬─────┘                     └────┬─────┘
                │                           ┌────┼────┐
                ▼ (pick best)               ▼    ▼    ▼  (try all)
          ┌──────────┐               ┌────┐ ┌────┐ ┌────┐
          │ Subprob  │               │ S1 │ │ S2 │ │ S3 │
          └────┬─────┘               └──┬─┘ └──┬─┘ └──┬─┘
               │                        │      │      │
               ▼ (pick best)            ▼      ▼      ▼
          ┌──────────┐               ┌────────────────────┐
          │ Subprob  │               │ Compare all, pick  │
          └────┬─────┘               │    the best one    │
               │                     └────────────────────┘
               ▼
           SOLUTION                      SOLUTION
        (fast, simple)              (slower, guaranteed)
```

| Aspect | Greedy | DP |
|--------|--------|----|
| Choices considered | One (locally best) | All possible |
| Reconsiders previous? | Never | Yes (tries everything) |
| Speed | Usually O(n log n) | Usually O(n^2) or O(n * W) |
| Correctness guarantee | Only if greedy property holds | Always correct (if recurrence is right) |
| Implementation | Simple — sort + one pass | More complex — table/memo |
| Memory | Usually O(1) or O(n) | Usually O(n) to O(n * W) |
| Think of it as | "Take the best now" | "What if I tried everything?" |

**Rule of thumb:** Try greedy first. If you can argue it is correct, you win — it is faster and simpler. If you cannot, fall back to [[Dynamic Programming]].

---

## 🔹 How to Prove Greedy Works

In interviews you rarely need a formal proof, but you should be able to **argue** why greedy is correct. Two standard techniques:

### Exchange Argument

1. Assume there exists an optimal solution `OPT` that does **not** use the greedy choice
2. Show you can **swap in** the greedy choice without making the solution worse
3. Therefore, there exists an optimal solution that **does** use the greedy choice

```
 OPT:    [ X | ... rest ... ]      ← doesn't use greedy choice G

 SWAP:   [ G | ... rest' ... ]     ← swap G in for X

 Show:   cost(SWAP) <= cost(OPT)
         ∴ greedy choice G is safe — using it never hurts

 Then apply inductively to the remaining subproblem.
```

### Stays-Ahead Argument

Show that at **every step**, the greedy solution is at least as good as any other:

```
 After step 1:  greedy >= any other
 After step 2:  greedy >= any other
 ...
 After step k:  greedy >= any other
 ∴ greedy is optimal at the end
```

**Interview tip:** You usually just need to say something like: *"We sort by end time because finishing earlier leaves the most room for future activities. Picking any other activity would finish later and could only reduce the number of remaining compatible activities."* That is enough — no formal proof needed.

---

## 🔹 Common Greedy Problems

---

### (a) Activity / Interval Selection

**Problem:** Given `n` activities, each with a start and end time, find the **maximum number of non-overlapping activities**.

**Greedy strategy:** Sort by **end time**. Always pick the activity that **finishes earliest** — this leaves the most room for future activities.

```
 Activities (unsorted):

 Timeline: 0   1   2   3   4   5   6   7   8   9  10  11  12
           |---|---|---|---|---|---|---|---|---|---|---|---|

 A: ████████████                  (0-4)
 B:     ████████████              (2-6)
 C:         ████████              (3-5)
 D:                 ████████      (5-7)
 E:                     ████████  (6-9)
 F:                         ████████████  (8-12)


 After sorting by END time:  A(0-4), C(3-5), B(2-6), D(5-7), E(6-9), F(8-12)

 GREEDY WALKTHROUGH:
 ┌───────┬────────────┬─────────────────────────────────────┐
 │ Step  │ Activity   │ Decision                            │
 ├───────┼────────────┼─────────────────────────────────────┤
 │  1    │ A(0-4)     │ PICK  (first one, always pick)      │
 │       │            │ last_end = 4                        │
 ├───────┼────────────┼─────────────────────────────────────┤
 │  2    │ C(3-5)     │ SKIP  (start 3 < last_end 4)       │
 ├───────┼────────────┼─────────────────────────────────────┤
 │  3    │ B(2-6)     │ SKIP  (start 2 < last_end 4)       │
 ├───────┼────────────┼─────────────────────────────────────┤
 │  4    │ D(5-7)     │ PICK  (start 5 >= last_end 4)      │
 │       │            │ last_end = 7                        │
 ├───────┼────────────┼─────────────────────────────────────┤
 │  5    │ E(6-9)     │ SKIP  (start 6 < last_end 7)       │
 ├───────┼────────────┼─────────────────────────────────────┤
 │  6    │ F(8-12)    │ PICK  (start 8 >= last_end 7)      │
 │       │            │ last_end = 12                       │
 └───────┴────────────┴─────────────────────────────────────┘

 Result on timeline:

   0   1   2   3   4   5   6   7   8   9  10  11  12
   ████████████         ████████     ████████████████
        A                   D               F

 Selected: {A, D, F} = 3 activities (OPTIMAL)
```

**Why sort by END time and not START time?**

```
 Sort by START time (WRONG):
   A: ██████████████████████  (0-10)  ← picked first, blocks everything
   B:   ████                  (1-3)
   C:       ████              (3-5)
   D:           ████          (5-7)
   Result: 1 activity (terrible!)

 Sort by END time (CORRECT):
   B:   ████                  (1-3)   ✓ PICK
   C:       ████              (3-5)   ✓ PICK
   D:           ████          (5-7)   ✓ PICK
   A: ██████████████████████  (0-10)  ✗ overlaps
   Result: 3 activities (optimal!)

 Sorting by end time ensures we never "waste" timeline space
 on a long activity when shorter ones could fit.
```

**C++ implementation:**

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

struct Activity {
    int start, end;
    int id;  // for display
};

vector<Activity> selectActivities(vector<Activity>& acts) {
    // Sort by end time — this IS the greedy strategy
    sort(acts.begin(), acts.end(), [](const Activity& a, const Activity& b) {
        return a.end < b.end;
    });

    vector<Activity> selected;
    int lastEnd = -1;  // end time of last selected activity

    for (const auto& act : acts) {
        if (act.start >= lastEnd) {   // no overlap with last picked
            selected.push_back(act);
            lastEnd = act.end;
        }
    }

    return selected;
}

int main() {
    vector<Activity> acts = {
        {0, 4, 1}, {2, 6, 2}, {3, 5, 3},
        {5, 7, 4}, {6, 9, 5}, {8, 12, 6}
    };

    auto result = selectActivities(acts);

    cout << "Maximum non-overlapping activities: " << result.size() << endl;
    for (const auto& a : result) {
        cout << "  Activity " << a.id
             << " [" << a.start << ", " << a.end << ")" << endl;
    }
    // Output:
    // Maximum non-overlapping activities: 3
    //   Activity 1 [0, 4)
    //   Activity 4 [5, 7)
    //   Activity 6 [8, 12)

    return 0;
}

// Time:  O(n log n) for sorting, O(n) for selection
// Space: O(n) for result
```

---

### (b) Fractional Knapsack

**Problem:** You have `n` items, each with a weight and value. Your knapsack has capacity `W`. You **CAN take fractions** of items. Maximize total value.

**Greedy strategy:** Sort by **value/weight ratio** (descending). Take as much of the highest-ratio item as possible, then move to the next.

```
 Example: Capacity W = 50

 ┌──────┬────────┬────────┬─────────────────┐
 │ Item │ Weight │ Value  │ Value/Weight     │
 ├──────┼────────┼────────┼─────────────────┤
 │  A   │  10    │  60    │ 6.0  ★ best     │
 │  B   │  20    │ 100    │ 5.0             │
 │  C   │  30    │ 120    │ 4.0             │
 └──────┴────────┴────────┴─────────────────┘

 GREEDY WALKTHROUGH:

 Step 1: Take ALL of A (weight 10, value 60)
   Knapsack: [AAAAAAAAAA........................]
   Capacity used: 10/50    Value so far: $60

 Step 2: Take ALL of B (weight 20, value 100)
   Knapsack: [AAAAAAAAAABBBBBBBBBBBBBBBBBBBB....]
   Capacity used: 30/50    Value so far: $160

 Step 3: Take 20/30 = 66.7% of C (weight 20, value 80)
   Knapsack: [AAAAAAAAAABBBBBBBBBBBBBBBBBBBBCCCC]
   Capacity used: 50/50    Value so far: $240

 ┌──────────────────────────────────────────────────┐
 │ Knapsack (capacity 50)                           │
 │ ┌────────┬──────────────────┬──────────────────┐ │
 │ │ A: 10  │ B: 20            │ C: 20 (of 30)    │ │
 │ │ $60    │ $100             │ $80              │ │
 │ └────────┴──────────────────┴──────────────────┘ │
 │                                                  │
 │ Total value: 60 + 100 + 80 = $240 (OPTIMAL)     │
 └──────────────────────────────────────────────────┘
```

**Fractional Knapsack vs 0/1 Knapsack:**

```
 FRACTIONAL (Greedy):         0/1 (needs DP!):
 ┌─────────────────────┐     ┌─────────────────────┐
 │ Can take 66.7%      │     │ Must take ALL or     │
 │ of an item          │     │ NOTHING of each item │
 │                     │     │                      │
 │ Highest ratio first │     │ Can't just grab best │
 │ always wins         │     │ ratio — might waste  │
 │                     │     │ capacity             │
 │ O(n log n)          │     │ O(n * W) with DP     │
 └─────────────────────┘     └─────────────────────┘

 Why 0/1 breaks greedy:
   Items: A(wt=10, val=60, ratio=6.0), B(wt=20, val=100, ratio=5.0)
   Capacity: 20

   Greedy picks A (ratio 6.0) → value $60, 10 capacity left
     Can't fit B → total = $60

   Optimal picks B → value $100
     Greedy missed this because it can't take a fraction of B!
```

See [[Common DP Patterns]] for the 0/1 Knapsack DP solution.

**C++ implementation:**

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

struct Item {
    double weight, value;
    double ratio() const { return value / weight; }
};

double fractionalKnapsack(vector<Item>& items, double capacity) {
    // Sort by value/weight ratio descending — this IS the greedy key
    sort(items.begin(), items.end(), [](const Item& a, const Item& b) {
        return a.ratio() > b.ratio();
    });

    double totalValue = 0.0;

    for (const auto& item : items) {
        if (capacity <= 0) break;

        if (item.weight <= capacity) {
            // Take the whole item
            totalValue += item.value;
            capacity -= item.weight;
            cout << "Take ALL of item (wt=" << item.weight
                 << ", val=" << item.value
                 << ", ratio=" << item.ratio() << ")" << endl;
        } else {
            // Take a fraction — fill remaining capacity
            double fraction = capacity / item.weight;
            totalValue += item.value * fraction;
            cout << "Take " << (fraction * 100) << "% of item (wt="
                 << item.weight << ", val=" << item.value
                 << ", gained=" << item.value * fraction << ")" << endl;
            capacity = 0;
        }
    }

    return totalValue;
}

int main() {
    vector<Item> items = {
        {10, 60},   // ratio 6.0
        {20, 100},  // ratio 5.0
        {30, 120}   // ratio 4.0
    };
    double W = 50;

    double maxVal = fractionalKnapsack(items, W);
    cout << "\nMaximum value: " << maxVal << endl;
    // Output:
    // Take ALL of item (wt=10, val=60, ratio=6)
    // Take ALL of item (wt=20, val=100, ratio=5)
    // Take 66.6667% of item (wt=30, val=120, gained=80)
    //
    // Maximum value: 240

    return 0;
}

// Time:  O(n log n) for sorting
// Space: O(1) extra
```

---

### (c) Huffman Coding (Conceptual Overview)

**Problem:** Assign **variable-length binary codes** to characters so that more frequent characters get **shorter** codes. This minimizes total encoding length.

**Greedy strategy:** Always merge the **two lowest-frequency** nodes into a new parent node. Repeat until one tree remains. Uses a [[Priority Queue]] (min-heap).

**Example:** Encode `"ABRACADABRA"` — A=5, B=2, R=2, C=1, D=1

```
 Character frequencies:
   A: █████  5
   B: ██     2
   R: ██     2
   C: █      1
   D: █      1

 Step-by-step tree construction (min-heap):

 Initial heap: [C(1), D(1), B(2), R(2), A(5)]

 Step 1: Merge C(1) + D(1) → node(2)
   Heap: [B(2), R(2), node(2), A(5)]

         (2)
        /   \
      C(1) D(1)

 Step 2: Merge B(2) + R(2) → node(4)
   Heap: [node(2), A(5), node(4)]

         (4)
        /   \
      B(2) R(2)

 Step 3: Merge node(2) + node(4) → node(6)
   Heap: [A(5), node(6)]

              (6)
             /   \
           (2)   (4)
          /   \ /   \
        C  D  B  R

 Step 4: Merge A(5) + node(6) → ROOT(11)

 Final Huffman Tree:

              [11]
             /    \
           0/      \1
           /        \
         A(5)      [6]
                  /    \
                0/      \1
                /        \
             [2]        [4]
            /    \     /    \
          0/      \1 0/      \1
          /        \ /        \
       C(1)     D(1) B(2)    R(2)


 Resulting codes (left=0, right=1):

 ┌───────┬──────┬───────┬─────────────────────────┐
 │ Char  │ Freq │ Code  │ Total bits (freq x len) │
 ├───────┼──────┼───────┼─────────────────────────┤
 │ A     │  5   │ 0     │  5 x 1 =  5            │
 │ C     │  1   │ 100   │  1 x 3 =  3            │
 │ D     │  1   │ 101   │  1 x 3 =  3            │
 │ B     │  2   │ 110   │  2 x 3 =  6            │
 │ R     │  2   │ 111   │  2 x 3 =  6            │
 └───────┴──────┴───────┴─────────────────────────┘
                          Total:     23 bits

 Comparison:
   Fixed-length (3 bits each): 11 chars x 3 bits = 33 bits
   Huffman encoding:           23 bits
   Savings:                    30% compression!
```

**Why greedy works:** Merging the **least frequent** nodes first pushes them deepest in the tree, giving them the longest codes — which is fine because they appear rarely. The most frequent character (A) ends up at the root with just a 1-bit code.

---

### (d) Jump Game

**Problem:** Array of non-negative integers where each element represents the max jump length from that position. Determine if you can reach the **last index** starting from index 0.

**Greedy strategy:** Track the **farthest reachable** position. Scan left to right — if your current index ever exceeds the farthest reachable, you are stuck.

```
 Example 1: nums = [2, 3, 1, 1, 4]   → TRUE

 Index:     0     1     2     3     4
 Value:    [2]   [3]   [1]   [1]   [4]

 i=0: val=2, farthest = max(0, 0+2) = 2
      Can reach: ├──────────┤
                 0     1    2

 i=1: val=3, farthest = max(2, 1+3) = 4   ← reached the end!
      Can reach: ├───────────────────────┤
                 0     1    2    3    4

 Result: TRUE (farthest=4 >= last index=4)


 Example 2: nums = [3, 2, 1, 0, 4]   → FALSE

 Index:     0     1     2     3     4
 Value:    [3]   [2]   [1]   [0]   [4]

 i=0: val=3, farthest = max(0, 0+3) = 3
      Can reach: ├────────────────┤
                 0     1    2    3

 i=1: val=2, farthest = max(3, 1+2) = 3  (no change)
 i=2: val=1, farthest = max(3, 2+1) = 3  (no change)
 i=3: val=0, farthest = max(3, 3+0) = 3  ← STUCK! val=0, can't advance
                                     ▲
                                     └── this zero is a "pit"

 i=4: i(4) > farthest(3)  → UNREACHABLE

 Result: FALSE

 Visualization of the "pit":
   [3] [2] [1] [0] [4]
    │   │   │   │   │
    ├───┤   │   │   │  can jump 3 from idx 0
    │   ├───┤   │   │  can jump 2 from idx 1
    │   │   ├───┤   │  can jump 1 from idx 2
    │   │   │   X   │  STUCK at idx 3 (jump = 0)
    │   │   │       │
    └───┴───┴───┘   │  farthest reachable = index 3
                    │
                   [4]  ← UNREACHABLE
```

**C++ implementation:**

```cpp
#include <iostream>
#include <vector>
using namespace std;

bool canJump(vector<int>& nums) {
    int farthest = 0;

    for (int i = 0; i < (int)nums.size(); i++) {
        if (i > farthest) return false;          // stuck — can't reach this index
        farthest = max(farthest, i + nums[i]);   // extend reach
        if (farthest >= (int)nums.size() - 1)
            return true;                         // early exit — can reach end
    }

    return true;
}

int main() {
    vector<int> nums1 = {2, 3, 1, 1, 4};
    cout << "Can jump [2,3,1,1,4]: "
         << (canJump(nums1) ? "YES" : "NO") << endl;
    // Output: YES

    vector<int> nums2 = {3, 2, 1, 0, 4};
    cout << "Can jump [3,2,1,0,4]: "
         << (canJump(nums2) ? "YES" : "NO") << endl;
    // Output: NO

    return 0;
}

// Time:  O(n) — single pass
// Space: O(1) — just one variable
```

---

### (e) Task Scheduler (LeetCode 621)

**Problem:** Given CPU tasks represented as characters and a cooldown period `n` between identical tasks, find the **minimum number of intervals** (including idle time) to finish all tasks.

**Greedy strategy:** The most frequent task dictates the schedule. Always schedule the **most frequent remaining task** first, filling cooldown gaps with other tasks or idle slots.

```
 Example: tasks = [A,A,A,B,B,B], cooldown n=2

 Most frequent: A and B, both appear 3 times
 maxFreq = 3,  maxFreqCount = 2

 Schedule pattern (n=2 means 2-slot gap between same tasks):

   Slot:  1   2   3   4   5   6   7   8
   Task:  A   B   _   A   B   _   A   B
          ↑       ↑   ↑       ↑   ↑
          |  cooldown |  cooldown |
          A───────────A───────────A   (gap of 2 between each A)
              B───────────B───────────B

 Framework visualization:

   (maxFreq-1) groups of (n+1) slots  +  final group

   [A  B  _] [A  B  _] [A  B]
   ╰───3───╯ ╰───3───╯ ╰──2─╯
    group 1    group 2   final (maxFreqCount tasks)

   Total = (maxFreq - 1) * (n + 1) + maxFreqCount
         = (3 - 1)       * (2 + 1) + 2
         = 2 * 3 + 2
         = 8
```

**Formula:**

```cpp
int leastInterval(vector<char>& tasks, int n) {
    int freq[26] = {};
    for (char c : tasks) freq[c - 'A']++;

    int maxFreq = *max_element(freq, freq + 26);
    int maxFreqCount = count(freq, freq + 26, maxFreq);

    return max((int)tasks.size(),
               (maxFreq - 1) * (n + 1) + maxFreqCount);
}
```

The `max` handles the case where there are so many diverse tasks that they fill all idle slots (total = tasks.size()).

---

## 🔹 When Greedy Does NOT Work

Not every problem that **looks** greedy actually is. These are the classic traps:

### Coin Change with Arbitrary Denominations

```
 Coins = {1, 3, 4},  Amount = 6

 ┌─────────────────────────────────────────────────────────┐
 │ GREEDY (pick largest first):                            │
 │                                                         │
 │   Remaining: 6  → pick 4  → remaining: 2               │
 │   Remaining: 2  → pick 1  → remaining: 1               │
 │   Remaining: 1  → pick 1  → remaining: 0               │
 │                                                         │
 │   Result: [4, 1, 1] = 3 coins   ✗ NOT optimal!         │
 │                                                         │
 ├─────────────────────────────────────────────────────────┤
 │ DP (try all combinations):                              │
 │                                                         │
 │   dp[0]=0, dp[1]=1, dp[2]=2, dp[3]=1, dp[4]=1,        │
 │   dp[5]=2, dp[6]=2                                     │
 │                                                         │
 │   Result: [3, 3] = 2 coins     ✓ OPTIMAL               │
 └─────────────────────────────────────────────────────────┘

 WHY greedy fails:
   Picking 4 first "blocks" us from the {3,3} combination.
   We would need to UNDO the choice of 4, but greedy never backtracks.
   The greedy choice (4) is locally best but globally suboptimal.
```

Need [[Dynamic Programming]] for general coin change — see [[Common DP Patterns]].

### 0/1 Knapsack

```
 Items:   A(wt=10, val=60, ratio=6.0)
          B(wt=20, val=100, ratio=5.0)
 Capacity: 20

 GREEDY (best ratio first):  Pick A → value $60, 10 left, B won't fit
                              Total: $60

 OPTIMAL (DP):               Pick B → value $100
                              Total: $100

 Greedy fails because we can't take fractions to "fill the gaps."
 The high-ratio item A wastes 10 units of capacity.
```

### General Rule — When Greedy Fails

```
 ┌──────────────────────────────────────────────────────────┐
 │  IF your problem requires any of these → DON'T use greedy│
 │                                                          │
 │  ✗ Considering multiple combinations of choices          │
 │  ✗ Undoing or reconsidering previous choices             │
 │  ✗ Counting ALL possible solutions                       │
 │  ✗ The locally best choice can block better future ones  │
 │                                                          │
 │  Classic "looks greedy but isn't":                       │
 │  ┌────────────────────────┬────────────────────────────┐ │
 │  │ GREEDY (works)         │ DP (greedy fails!)         │ │
 │  ├────────────────────────┼────────────────────────────┤ │
 │  │ Fractional Knapsack    │ 0/1 Knapsack               │ │
 │  │ Coin Change (US coins) │ Coin Change (arbitrary)     │ │
 │  │ Activity Selection     │ Weighted Job Scheduling     │ │
 │  │ Jump Game (can reach?) │ Jump Game II (min jumps)    │ │
 │  │ Huffman Coding         │ Optimal BST                 │ │
 │  └────────────────────────┴────────────────────────────┘ │
 │                                                          │
 │  The difference is often ONE WORD that changes approach! │
 └──────────────────────────────────────────────────────────┘
```

---

## 🔹 Decision Flowchart — When to Use What

```
          ┌──────────────────────────────────────┐
          │  Can you make irrevocable             │
          │  locally-optimal choices?             │
          └──────────┬──────────┬────────────────┘
                     │          │
                    YES         NO
                     │          │
                     ▼          │
          ┌──────────────────┐  │
          │ Does it provably  │  │
          │ lead to global    │  │
          │ optimum?          │  │
          └────┬────────┬────┘  │
               │        │      │
              YES       NO     │
               │        │      │
               ▼        ▼      ▼
          ┌────────┐ ┌──────────────────────────────┐
          │ GREEDY │ │ Need DP or Backtracking       │
          │        │ │                                │
          │ Fast!  │ │ Overlapping subproblems?       │
          │ Simple!│ │   YES → [[Dynamic Programming]]│
          └────────┘ │   NO  → [[Backtracking]]       │
                     │                                │
                     │ Need ALL valid solutions?       │
                     │   YES → [[Backtracking]]        │
                     │         + [[Recursion]]          │
                     └──────────────────────────────┘

 ┌─────────────────────────┬──────────┬──────────────────────┐
 │ Problem Trait            │ Approach │ Why                  │
 ├─────────────────────────┼──────────┼──────────────────────┤
 │ Locally best = globally │ Greedy   │ No need to explore   │
 │ best, no take-backs     │          │ all possibilities    │
 ├─────────────────────────┼──────────┼──────────────────────┤
 │ Overlapping subproblems,│ DP       │ Cache saves repeated │
 │ optimal value needed    │          │ recomputation        │
 ├─────────────────────────┼──────────┼──────────────────────┤
 │ Need ALL solutions or   │ Backtrack│ Must explore the     │
 │ constraints are complex │          │ full search tree     │
 └─────────────────────────┴──────────┴──────────────────────┘
```

---

## 🔹 Common Pitfalls

### 1. Assuming Greedy Works Without Verification

```
 "It feels greedy..."     ← NOT a proof
 "Sort and pick best..."  ← MIGHT be wrong

 ALWAYS ask yourself:
   "Can the locally best choice BLOCK a better future choice?"

   YES → greedy fails → use [[Dynamic Programming]]
   NO  → greedy is safe (but test counterexamples!)
```

### 2. Wrong Sort Key Ruins Everything

The sort key **IS** the greedy strategy. Get it wrong and the entire algorithm produces garbage.

```
 Activity Selection:
   Sort by START time  → WRONG (long early activity blocks all)
   Sort by DURATION    → WRONG (short but poorly placed)
   Sort by END time    → CORRECT

 Fractional Knapsack:
   Sort by value alone → WRONG (heavy expensive item wastes capacity)
   Sort by weight alone → WRONG (light cheap item wastes value)
   Sort by value/weight → CORRECT
```

### 3. Confusing Problem Variants

```
 THE GREEDY TRAP:

   "Fractional Knapsack is greedy,
    so 0/1 Knapsack must be too!"  ← WRONG!

   "Coin change works greedily with US coins,
    so it works with any coins!"   ← WRONG!

   "Jump Game I is greedy,
    so Jump Game II must be too!"  ← NEEDS MORE THOUGHT

 EACH VARIANT must be analyzed independently.
 One word in the problem statement can change the approach entirely.
```

---

## 🔹 Summary

```
 ┌──────────────────────────────────────────────────────────┐
 │                   GREEDY TECHNIQUE                       │
 │                                                          │
 │  Core:   Make the best choice NOW, never look back,      │
 │          and hope it leads to the globally best answer.   │
 │                                                          │
 │  When it works  → elegant, fast, simple (O(n log n))     │
 │  When it doesn't → WRONG answer, must use DP instead     │
 │                                                          │
 │  Two required properties:                                │
 │    1. Greedy Choice Property (local → global)            │
 │    2. Optimal Substructure (shared with DP)               │
 │                                                          │
 │  Classic greedy problems:                                │
 │    ✓ Activity Selection     (sort by end time)           │
 │    ✓ Fractional Knapsack    (sort by value/weight ratio) │
 │    ✓ Huffman Coding         (merge smallest freq first)  │
 │    ✓ Jump Game              (track farthest reachable)   │
 │    ✓ Task Scheduler         (schedule most frequent)     │
 │                                                          │
 │  NOT greedy (need DP):                                   │
 │    ✗ 0/1 Knapsack                                        │
 │    ✗ Coin Change (arbitrary denominations)                │
 │    ✗ Weighted Job Scheduling                              │
 │                                                          │
 │  Related:                                                │
 │    [[Dynamic Programming]]   [[Common DP Patterns]]      │
 │    [[Memoization vs Tabulation]]  [[Backtracking]]       │
 │    [[Recursion]]   [[Priority Queue]]                    │
 └──────────────────────────────────────────────────────────┘
```
