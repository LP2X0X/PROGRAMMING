---
tags:
  - algorithms
  - pattern-recognition
  - two-pointers
---

## 🔹 When to Suspect This Pattern

- Input is a **sorted** array or linked list
- Keywords: "pair with sum", "triplet", "palindrome", "in-place removal", "merge two sorted"
- Need to find **two elements** satisfying a condition (sum, difference, product)
- Need to **partition** or **rearrange** an array in-place
- Problem involves **comparing from both ends** simultaneously
- **Cycle detection** in a linked list

---

## 🔹 Confirming It's the Right Pattern

- [ ] Is the input sorted (or can sorting it help)?
- [ ] Are you looking for a pair/triplet with a target relationship?
- [ ] Can you make progress by moving one of two positions inward?
- [ ] Is the problem asking for in-place modification with O(1) space?
- [ ] Does moving one pointer give you information about which direction to go?

---

## 🔹 Three Variants

### Variant 1: Opposite Direction (Converging)

Pointers start at both ends, move inward. Used for sorted arrays.

```
[1, 2, 3, 5, 8, 11, 15]
 ^                    ^
 left              right
```

```cpp
int left = 0, right = n - 1;
while (left < right) {
    int sum = arr[left] + arr[right];
    if (sum == target) return {left, right};
    else if (sum < target) left++;
    else right--;
}
```

**Use for**: Two Sum (sorted), Container With Most Water, Trapping Rain Water, palindrome check.

### Variant 2: Same Direction (Fast/Slow or Read/Write)

Both pointers move in the same direction at different speeds.

```cpp
// Remove duplicates in-place (sorted array)
int write = 1;
for (int read = 1; read < n; read++) {
    if (arr[read] != arr[read - 1])
        arr[write++] = arr[read];
}
// arr[0..write-1] contains unique elements
```

**Use for**: Remove duplicates, remove element, move zeroes, partition array.

### Variant 3: Fast/Slow (Floyd's Cycle Detection)

Two pointers moving at different speeds through a linked list.

```cpp
ListNode* slow = head;
ListNode* fast = head;
while (fast && fast->next) {
    slow = slow->next;
    fast = fast->next->next;
    if (slow == fast) return true;  // cycle found
}
return false;
```

**Use for**: Detect cycle, find cycle start, find middle of linked list, happy number.

---

## 🔹 Classic Problems

| Problem | Variant | Key Insight |
|---|---|---|
| **Two Sum II (sorted)** | Opposite | Sum too small → move left; too large → move right |
| **3Sum** | Sort + Opposite | Fix one element, two-pointer for remaining pair |
| **Container With Most Water** | Opposite | Move the shorter side inward (can only get better) |
| **Remove Duplicates** | Same direction (read/write) | Write pointer skips duplicates |
| **Move Zeroes** | Same direction (read/write) | Write pointer places non-zeros |
| **Palindrome Check** | Opposite | Compare from both ends toward center |
| **Linked List Cycle** | Fast/Slow | Fast moves 2x; if cycle, they meet |
| **Middle of Linked List** | Fast/Slow | When fast reaches end, slow is at middle |
| **Merge Two Sorted Arrays** | Same direction (two arrays) | Compare heads, take smaller |

---

## 🔹 Template: 3Sum Using Sort + Two Pointers

```cpp
sort(arr.begin(), arr.end());
vector<vector<int>> result;
for (int i = 0; i < n - 2; i++) {
    if (i > 0 && arr[i] == arr[i-1]) continue;  // skip duplicates
    int left = i + 1, right = n - 1;
    while (left < right) {
        int sum = arr[i] + arr[left] + arr[right];
        if (sum == 0) {
            result.push_back({arr[i], arr[left], arr[right]});
            while (left < right && arr[left] == arr[left+1]) left++;
            while (left < right && arr[right] == arr[right-1]) right--;
            left++; right--;
        } else if (sum < 0) left++;
        else right--;
    }
}
```

---

## 🔹 Common Mistakes

- **Forgetting to sort first**: two pointers on unsorted data gives wrong results (use hash map instead for unsorted)
- **Duplicate handling in 3Sum**: must skip duplicates for all three positions, not just one
- **Off-by-one on `while (left < right)` vs `while (left <= right)`**: for pairs, use `<`; for single element search, use `<=`
- **Fast/slow pointer: checking `fast->next` before `fast->next->next`**: always check `fast && fast->next`
- **Assuming two pointers always works**: only works when you can guarantee which pointer to move. If there's no monotonic relationship, two pointers won't help

---

## 🔹 Related Patterns

- [[Pattern - Sliding Window]] — same-direction pointers with a window (specialized variant)
- [[Pattern - Hash Map]] — alternative when input is unsorted
- [[Pattern - Binary Search]] — alternative search on sorted data
- [[Pattern - Array and String]] — two pointers for in-place string manipulation
