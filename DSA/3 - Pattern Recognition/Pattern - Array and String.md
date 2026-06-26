---
tags:
  - algorithms
  - pattern-recognition
  - array
  - string
---

## 🔹 When to Suspect This Pattern

- Problem operates on **sequential data** (array or string)
- Keywords: "subarray", "substring", "prefix", "suffix", "in-place", "contiguous"
- Need to compute **running totals**, cumulative sums, or range sums
- Problem asks for **maximum/minimum subarray** value
- String problems involving **pattern matching**, reversals, or rotations
- "Modify the array in-place" or "use O(1) extra space"

---

## 🔹 Key Sub-Patterns

### Prefix Sum

Use when you need **range sum queries** or "subarray sum equals K."

| Signal | Example |
|---|---|
| "Sum of elements from index i to j" | Range sum query |
| "Number of subarrays with sum = K" | Subarray Sum Equals K |
| "Balance point / pivot index" | Pivot Index |

**Template**:
```cpp
// Build prefix sum
vector<int> prefix(n + 1, 0);
for (int i = 0; i < n; i++)
    prefix[i + 1] = prefix[i] + arr[i];

// Query sum of arr[l..r] (inclusive)
int rangeSum = prefix[r + 1] - prefix[l];
```

**With Hash Map** (subarray sum = K):
```cpp
unordered_map<int, int> prefixCount;
prefixCount[0] = 1;  // empty prefix
int sum = 0, count = 0;
for (int x : arr) {
    sum += x;
    if (prefixCount.count(sum - k))
        count += prefixCount[sum - k];
    prefixCount[sum]++;
}
```

### Kadane's Algorithm (Maximum Subarray)

Use when the problem asks for the **maximum (or minimum) sum contiguous subarray**.

**Signal**: "maximum subarray sum", "largest contiguous sum"

```cpp
int maxSum = arr[0], current = arr[0];
for (int i = 1; i < n; i++) {
    current = max(arr[i], current + arr[i]);  // extend or restart
    maxSum = max(maxSum, current);
}
```

> [!tip] Kadane's Core Insight
> At each position, decide: is it better to **extend** the current subarray or **start fresh** here? That's the entire algorithm.

### String Matching / Comparison

| Technique | When to Use |
|---|---|
| Two Pointers | Palindrome check, comparing from both ends |
| Hash Map | Anagram detection, character frequency |
| Sliding Window | Substring with constraints |
| KMP / Z-Algorithm | Pattern matching in O(n + m) — for large strings |
| Rolling Hash (Rabin-Karp) | Multiple pattern matching |

---

## 🔹 Confirming It's the Right Pattern

- [ ] Is the data sequential (array or string)?
- [ ] Does the problem involve **contiguous elements** (not arbitrary selections)?
- [ ] Can a prefix sum, running computation, or scan solve it?
- [ ] If string: is it about characters, substrings, or pattern matching?

---

## 🔹 Classic Problems

| Problem | Sub-Pattern | Key Idea |
|---|---|---|
| Maximum Subarray | Kadane's | Extend or restart at each step |
| Subarray Sum Equals K | Prefix Sum + Hash Map | Store prefix sums, check `sum - k` |
| Product of Array Except Self | Prefix/Suffix Product | Left pass * right pass |
| Rotate Array | Reversal trick | Reverse all, reverse first k, reverse rest |
| Longest Common Prefix | Vertical scan | Compare char-by-char across all strings |

---

## 🔹 Common Mistakes

- **Prefix sum off-by-one**: prefix array is size `n+1`, prefix[0] = 0. Range `[l, r]` = `prefix[r+1] - prefix[l]`
- **Kadane's with all negatives**: initialize `maxSum = arr[0]`, not `0` or `INT_MIN` carelessly
- **Forgetting empty subarray**: some problems allow empty subarray (sum = 0); Kadane's default doesn't
- **String immutability**: in some languages strings are immutable — convert to `char[]` for in-place ops
- **Integer overflow**: prefix sums on large arrays with large values — use `long long`

---

## 🔹 Related Patterns

- [[Pattern - Sliding Window]] — for "contiguous subarray/substring with constraint"
- [[Pattern - Two Pointers]] — for in-place modifications, palindrome checks
- [[Pattern - Hash Map]] — often combined with prefix sum
- [[Pattern - Dynamic Programming]] — when subarray problems have overlapping subproblems
