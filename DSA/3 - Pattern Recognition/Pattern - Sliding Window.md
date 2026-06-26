---
tags:
  - algorithms
  - pattern-recognition
  - sliding-window
---

## 🔹 When to Suspect This Pattern

- Problem involves a **contiguous subarray or substring**
- Keywords: "maximum/minimum of size K", "longest/shortest with condition", "at most K distinct", "subarray with sum >= target"
- You need to find an **optimal contiguous segment** under some constraint
- Brute force would check **all subarrays** — O(n^2) or worse
- The constraint is **monotonic**: expanding the window can only violate (or only satisfy) the condition in one direction

---

## 🔹 Confirming It's the Right Pattern

- [ ] Is the answer a **contiguous** subarray or substring?
- [ ] Is there a **constraint** on the window (size, sum, distinct count, etc.)?
- [ ] When the window is invalid, does **shrinking from the left** fix it?
- [ ] When the window is valid, does **expanding from the right** keep exploring?
- [ ] Can each element be added/removed from the window in O(1)?

> [!warning] Sliding Window Doesn't Work When...
> - Elements can be negative AND you're tracking sum constraints (shrinking doesn't reliably help). Use prefix sum + hash map instead (see [[Pattern - Array and String]])
> - The subarray is not contiguous (e.g., subsequence problems → use [[Pattern - Dynamic Programming]])

---

## 🔹 Two Variants

### Fixed-Size Window (Size K)

The window size is given. Slide it across and compute something for each position.

```cpp
// Maximum sum of subarray of size K
int windowSum = 0;
for (int i = 0; i < k; i++)
    windowSum += arr[i];

int maxSum = windowSum;
for (int i = k; i < n; i++) {
    windowSum += arr[i] - arr[i - k];  // slide: add right, remove left
    maxSum = max(maxSum, windowSum);
}
```

**Use for**: Maximum/average of size K, fixed-size substring matching.

### Variable-Size Window (Shrink/Expand)

Window size changes to satisfy a constraint. The template:

```cpp
int left = 0;
int best = 0;  // or INT_MAX for "shortest"

for (int right = 0; right < n; right++) {
    // 1. EXPAND: add arr[right] to window state

    // 2. SHRINK: while window is invalid, remove arr[left]
    while (window_is_invalid()) {
        // remove arr[left] from window state
        left++;
    }

    // 3. UPDATE: window [left..right] is now valid
    best = max(best, right - left + 1);  // longest
    // or: best = min(best, right - left + 1);  // shortest (update inside while loop instead)
}
```

> [!tip] Longest vs Shortest
> - **Longest valid**: shrink until valid, update answer **after** the while loop
> - **Shortest valid**: update answer **inside** the while loop (each time window is valid before shrinking further)

---

## 🔹 Template: Shortest Subarray with Sum >= Target

```cpp
int left = 0, sum = 0, minLen = INT_MAX;
for (int right = 0; right < n; right++) {
    sum += arr[right];
    while (sum >= target) {
        minLen = min(minLen, right - left + 1);  // update inside: shortest
        sum -= arr[left++];
    }
}
```

### Template: Longest Substring with At Most K Distinct Characters

```cpp
unordered_map<char, int> freq;
int left = 0, best = 0;
for (int right = 0; right < n; right++) {
    freq[s[right]]++;
    while (freq.size() > k) {
        freq[s[left]]--;
        if (freq[s[left]] == 0) freq.erase(s[left]);
        left++;
    }
    best = max(best, right - left + 1);  // update outside: longest
}
```

---

## 🔹 Classic Problems

| Problem | Window Type | Constraint |
|---|---|---|
| **Max Sum Subarray of Size K** | Fixed | Size = K |
| **Minimum Size Subarray Sum** | Variable (shortest) | Sum >= target |
| **Longest Substring Without Repeating** | Variable (longest) | All chars unique |
| **Longest Substring with At Most K Distinct** | Variable (longest) | Distinct count <= K |
| **Minimum Window Substring** | Variable (shortest) | Contains all target chars |
| **Permutation in String / Find Anagram** | Fixed | Frequency matches target |
| **Max Consecutive Ones III** | Variable (longest) | At most K zeros flipped |

---

## 🔹 Common Mistakes

- **Using sliding window with negative numbers for sum problems**: shrinking may not help. Use prefix sum approach instead
- **Updating answer at wrong location**: longest → after while loop; shortest → inside while loop
- **Forgetting to clean up the map**: when a char's frequency drops to 0, erase it if you're checking `map.size()` for distinct count
- **Off-by-one on window size**: window `[left, right]` has size `right - left + 1`
- **Not recognizing it as sliding window**: "longest substring" / "minimum subarray" are the strongest signals

---

## 🔹 Related Patterns

- [[Pattern - Two Pointers]] — sliding window is a specialized same-direction two pointers
- [[Pattern - Hash Map]] — often used inside the window to track frequencies
- [[Pattern - Array and String]] — prefix sum is the alternative when sliding window doesn't apply
