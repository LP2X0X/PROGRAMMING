---
tags:
  - algorithms
  - pattern-recognition
  - binary-search
---

## 🔹 When to Suspect This Pattern

- Input is **sorted** (or has a monotonic property)
- Keywords: "find", "search", "minimum that satisfies", "maximum that satisfies"
- The **answer space** can be halved at each step
- "Binary search on the answer" — the answer has a **monotonic feasibility property**
- Need to find a **boundary** (first true, last false in a boolean array)
- Complexity requirement is **O(log n)**

---

## 🔹 Confirming It's the Right Pattern

- [ ] Is the search space **sorted** or **monotonic**?
- [ ] Can you determine which **half to discard** based on a comparison?
- [ ] For "search on answer": is the feasibility function monotonic? (If X works, does X+1 also work? Or vice versa?)
- [ ] Is O(log n) required by the constraints? (see [[Constraint Analysis]])

---

## 🔹 Three Main Variants

### Variant 1: Classic Binary Search (Find Exact Value)

```cpp
int left = 0, right = n - 1;
while (left <= right) {
    int mid = left + (right - left) / 2;  // avoid overflow
    if (arr[mid] == target) return mid;
    else if (arr[mid] < target) left = mid + 1;
    else right = mid - 1;
}
return -1;  // not found
```

### Variant 2: Find Boundary (First/Last Position)

**First position where condition is true** (lower bound):

```cpp
int left = 0, right = n;  // right = n, not n-1
while (left < right) {     // < not <=
    int mid = left + (right - left) / 2;
    if (condition(mid))
        right = mid;       // mid might be the answer
    else
        left = mid + 1;    // mid is definitely not the answer
}
return left;  // first position where condition is true
```

**Last position where condition is true** (upper bound):

```cpp
int left = 0, right = n - 1;
while (left < right) {
    int mid = left + (right - left + 1) / 2;  // round UP to avoid infinite loop
    if (condition(mid))
        left = mid;        // mid might be the answer
    else
        right = mid - 1;
}
return left;
```

> [!warning] Infinite Loop Trap
> When searching for the **last** position (`left = mid`), you must round **up**: `mid = left + (right - left + 1) / 2`. Otherwise when `left + 1 == right`, `mid = left` and the loop never ends.

### Variant 3: Binary Search on the Answer

When the answer itself is the search space, and you can check feasibility.

```cpp
// "Minimize the maximum" or "maximize the minimum"
int lo = min_possible_answer;
int hi = max_possible_answer;

while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (isFeasible(mid))
        hi = mid;       // mid works, try smaller
    else
        lo = mid + 1;   // mid doesn't work, need larger
}
return lo;
```

**Examples**: 
- "Minimize maximum distance between gas stations"
- "Split array into K subarrays minimizing max sum"
- "Koko eating bananas — minimum speed"

---

## 🔹 Classic Problems

| Problem | Variant | Key Insight |
|---|---|---|
| **Binary Search** | Classic | Exact match in sorted array |
| **First/Last Position in Sorted Array** | Boundary | Two binary searches: lower bound + upper bound |
| **Search in Rotated Sorted Array** | Modified classic | One half is always sorted; determine which |
| **Find Peak Element** | Boundary | Compare `mid` with `mid+1` to determine direction |
| **Koko Eating Bananas** | Search on answer | Binary search on speed; check if feasible in H hours |
| **Split Array Largest Sum** | Search on answer | Binary search on max sum; greedy check feasibility |
| **Median of Two Sorted Arrays** | Partition-based | Binary search on partition position |
| **Search a 2D Matrix** | Classic | Treat 2D as 1D with index math |

---

## 🔹 The Binary Search Template Decision

| Question | `while (left <= right)` | `while (left < right)` |
|---|---|---|
| Looking for exact match? | **Yes** | No — use for boundaries |
| Searching for boundary? | No (error-prone) | **Yes** |
| What does `left` mean at end? | Past-the-answer (for boundaries) | The answer itself |
| Risk of infinite loop? | No | Yes, if not rounding correctly |

---

## 🔹 Common Mistakes

- **Integer overflow**: `(left + right) / 2` overflows. Use `left + (right - left) / 2`
- **Off-by-one on boundaries**: should `right` start at `n` or `n-1`? Depends on whether the answer can be "past the end"
- **Wrong loop condition**: `<=` for exact search, `<` for boundary search
- **Not handling "not found"**: classic search returns -1; boundary search returns `left` which may equal `n` (no valid answer)
- **Failing to verify monotonicity** for "search on answer": if feasibility isn't monotonic, binary search gives wrong results
- **Rotated array**: forgetting to check which half is sorted before deciding direction

---

## 🔹 Related Patterns

- [[Pattern - Two Pointers]] — alternative for sorted array problems (pair search)
- [[Pattern - Dynamic Programming]] — binary search + DP (LIS in O(n log n))
- [[Constraint Analysis]] — when n > 10^8, binary search is likely needed
- [[How to Pick the Right Algorithm]] — binary search vs linear scan
