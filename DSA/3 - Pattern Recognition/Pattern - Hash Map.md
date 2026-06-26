---
tags:
  - algorithms
  - pattern-recognition
  - hash-map
---

## 🔹 When to Suspect This Pattern

- Need **O(1) lookup** — "check if X exists"
- **Frequency counting** — "how many times does X appear?"
- **Finding pairs or groups** — "two elements that sum to K"
- Keywords: "duplicate", "anagram", "group", "frequency", "count", "unique"
- Need to **map** one value to another (index, complement, grouping key)
- Problem can be solved by "remembering what you've seen so far"

---

## 🔹 Confirming It's the Right Pattern

- [ ] Do you need O(1) lookup, insert, or delete?
- [ ] Are you counting occurrences or checking existence?
- [ ] Can the problem be restated as "for each element, have I seen its complement?"
- [ ] Do you need to group elements by some property?
- [ ] Would brute force involve nested loops where the inner loop is just "searching"?

> [!tip] The Hash Map Test
> If your brute force is O(n^2) because of a nested search loop, a hash map almost always brings it to O(n) by replacing that inner loop with O(1) lookup.

---

## 🔹 Template Approaches

### Pattern 1: Complement Lookup (Two Sum style)

```cpp
unordered_map<int, int> seen;  // value → index
for (int i = 0; i < n; i++) {
    int complement = target - arr[i];
    if (seen.count(complement))
        return {seen[complement], i};
    seen[arr[i]] = i;
}
```

### Pattern 2: Frequency Count

```cpp
unordered_map<char, int> freq;
for (char c : s)
    freq[c]++;

// Check: are two strings anagrams?
// Build freq for s1, decrement for s2, check all zeros
```

### Pattern 3: Grouping by Key

```cpp
unordered_map<string, vector<string>> groups;
for (string& word : words) {
    string key = sorted(word);  // or any canonical form
    groups[key].push_back(word);
}
// groups now contains anagram groups
```

### Pattern 4: First/Last Occurrence Tracking

```cpp
unordered_map<int, int> firstSeen;  // value → first index
for (int i = 0; i < n; i++) {
    if (!firstSeen.count(arr[i]))
        firstSeen[arr[i]] = i;
}
```

---

## 🔹 Classic Problems

| Problem | How Hash Map Is Used |
|---|---|
| **Two Sum** | Store complement → index; check if complement exists |
| **Group Anagrams** | Sort each word as key; group by key |
| **Longest Substring Without Repeating** | Map char → last index; track window start |
| **Subarray Sum Equals K** | Prefix sum → count; check `sum - k` (see [[Pattern - Array and String]]) |
| **Valid Anagram** | Frequency map of both strings; compare |
| **Contains Duplicate** | Hash set — O(1) existence check |
| **Top K Frequent Elements** | Frequency map + bucket sort or heap |

---

## 🔹 Common Mistakes

- **Using map instead of unordered_map** (C++): `map` is O(log n) per operation (red-black tree). Use `unordered_map` for O(1) average. Only use `map` if you need sorted keys
- **Hash collisions**: worst case O(n) per operation. Rare in practice, but be aware in adversarial inputs
- **Forgetting to initialize**: checking `map[key]` in C++ auto-inserts with default value. Use `map.count(key)` or `map.find(key)` to check without inserting
- **Mutable keys**: never use a mutable object as a hash key
- **Space overhead**: hash maps use more memory than arrays. If keys are small integers (0 to n), use an array instead

> [!warning] When NOT to Use Hash Map
> - If you need **sorted order** → use BST (`std::map`) or sorted array
> - If keys are **integers in small range** → use a plain array (faster, less memory)
> - If you need **range queries** → hash map can't do this; use BST or segment tree

---

## 🔹 Hash Map vs Hash Set

| Use Case | Structure | What You Store |
|---|---|---|
| "Does X exist?" | Hash **Set** | Just the element |
| "What is X mapped to?" | Hash **Map** | Key-value pair |
| "How many X?" | Hash **Map** | Element → count |
| "Remove duplicates" | Hash **Set** | Unique elements |

---

## 🔹 Related Patterns

- [[Pattern - Array and String]] — prefix sum + hash map is a classic combo
- [[Pattern - Sliding Window]] — hash map tracks frequency within the window
- [[Pattern - Two Pointers]] — alternative to hash map when data is sorted
- [[How to Pick the Right Data Structure]] — when to use hash map vs BST vs array
