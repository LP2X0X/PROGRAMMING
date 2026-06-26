---
tags:
  - algorithms
  - data-structure
  - hash-table
---

## 🔹 Real-World Analogy

Imagine a library where every book is stored on a random shelf. To find a specific book, you would have to scan every shelf -- O(n) time. Now imagine the library has a system: take the book's title, run it through a formula, and it tells you *exactly* which shelf to go to. You walk straight there -- O(1) time.

That is what a hash table does. It transforms a **key** (like a name, word, or number) into a **direct address** in an array, giving you near-instant lookup.

Other analogies:
- **Phone book**: you don't scan every entry -- you jump to the letter section, then narrow down.
- **Dictionary**: you look up "ephemeral" by going to the "E" section, not by reading from page 1.
- **Coat check**: you hand in your coat (value), get a numbered ticket (hash), and retrieve it instantly later.

The key insight: **hash tables trade space for speed**. They use extra memory (an underlying array) to achieve O(1) average-case operations.

---

## 🔹 What Is Hashing?

A **hash function** takes an input (a key) and produces a fixed-size integer output (a hash code). This integer is then used as an index into an array.

```
key  --->  hash function  --->  integer  --->  index = integer % table_size
```

### Example

```
hash("apple")  = 394829107
table_size     = 10

index = 394829107 % 10 = 7

So "apple" is stored at bucket index 7.
```

### Properties of a Good Hash Function

| Property                 | Why It Matters                                         |
| ------------------------ | ------------------------------------------------------ |
| **Deterministic**        | Same key must always produce the same hash             |
| **Uniform distribution** | Keys should spread evenly across all buckets           |
| **Fast to compute**      | The whole point is O(1) -- a slow hash defeats that    |
| **Minimizes collisions** | Different keys should (ideally) produce different hashes |
| **Avalanche effect**     | Small change in input should drastically change output |

A poor hash function (e.g., always returning 0) would put every key into the same bucket, degrading the hash table to a linked list -- O(n) for everything.

---

## 🔹 Hash Map vs Hash Set

These are the two fundamental hash-table-based data structures. They share the same underlying mechanism but serve different purposes.

| Feature         | Hash Map                            | Hash Set                            |
| --------------- | ----------------------------------- | ----------------------------------- |
| Stores          | **Key-value pairs**                 | **Keys only** (no associated value) |
| Purpose         | Associate data with keys            | Track membership / uniqueness       |
| Lookup question | "What value is associated with X?"  | "Does X exist in the set?"          |
| Example         | `{"apple": 5, "banana": 3}`        | `{"apple", "banana", "cherry"}`     |
| C++ STL         | `std::unordered_map<K,V>`           | `std::unordered_set<K>`             |
| C#              | `Dictionary<K,V>`                   | `HashSet<T>`                        |
| Java            | `HashMap<K,V>`                      | `HashSet<T>`                        |
| Python          | `dict`                              | `set`                               |

**Mental model**: A Hash Set is just a Hash Map where the value is irrelevant (or implicitly `true`). Internally, many implementations literally use a hash map with a dummy value.

When to use which:
- **Hash Map**: "I need to look up a value by its key" -- frequency counts, caching, two-sum
- **Hash Set**: "I only care whether something exists" -- deduplication, cycle detection, visited tracking

---

## 🔹 Internal Structure

A hash table is fundamentally an **[[Arrays|array]] of buckets**. Each bucket can hold one or more entries. The hash function decides which bucket a key goes into.

### ASCII Art: Hash Table Structure

```
           Hash Function
Key ──────────┐
              ▼
         ┌─────────┐
         │ hash(key)│
         │  % size  │
         └────┬─────┘
              │  index
              ▼
   Buckets (Array)
   ┌───┐
 0 │   │ → empty
   ├───┤
 1 │   │ → empty
   ├───┤
 2 │   │ → ("banana", 3)
   ├───┤
 3 │   │ → empty
   ├───┤
 4 │   │ → ("cherry", 7) → ("grape", 2)    ← collision! chained
   ├───┤
 5 │   │ → empty
   ├───┤
 6 │   │ → empty
   ├───┤
 7 │   │ → ("apple", 5)
   └───┘
```

### Step-by-Step: Inserting Three Entries

Let's insert `"apple"→5`, `"banana"→3`, `"cherry"→7` into a table of size 8.

**Step 1: Insert "apple" → 5**
```
hash("apple") = 394829107
index = 394829107 % 8 = 3

Buckets:
 0 [ ]
 1 [ ]
 2 [ ]
 3 [ ("apple", 5) ]  ← inserted here
 4 [ ]
 5 [ ]
 6 [ ]
 7 [ ]
```

**Step 2: Insert "banana" → 3**
```
hash("banana") = 762184501
index = 762184501 % 8 = 5

Buckets:
 0 [ ]
 1 [ ]
 2 [ ]
 3 [ ("apple", 5) ]
 4 [ ]
 5 [ ("banana", 3) ]  ← inserted here
 6 [ ]
 7 [ ]
```

**Step 3: Insert "cherry" → 7**
```
hash("cherry") = 228491683
index = 228491683 % 8 = 3   ← COLLISION with "apple"!

Buckets:
 0 [ ]
 1 [ ]
 2 [ ]
 3 [ ("apple", 5) ] → [ ("cherry", 7) ]  ← chained after "apple"
 4 [ ]
 5 [ ("banana", 3) ]
 6 [ ]
 7 [ ]
```

When "cherry" hashes to the same bucket as "apple", this is a **collision**. The most common resolution is **chaining** -- storing multiple entries in the same bucket using a [[Singly Linked List|linked list]] (or in modern implementations, a balanced [[Trees|tree]] once the chain gets long).

See: [[Collision Handling]] for detailed coverage of chaining vs open addressing.

### Load Factor

```
load_factor = n / table_size

where n = number of stored entries
```

- When load factor exceeds a threshold (typically 0.75), the table **resizes** (usually doubles) and rehashes all entries.
- This keeps chains short and maintains O(1) average performance.
- Resizing itself is O(n), but it happens infrequently enough that the **amortized** cost of insertion remains O(1).

---

## 🔹 Operations and Complexity

### Insert (Put)

```cpp
void insert(Key k, Value v) {
    int idx = hash(k) % table_size;
    // Walk the chain at buckets[idx]
    // If key already exists, update the value
    // Otherwise, append new (k, v) to the chain
}
```

- **Average O(1)**: hash is computed in O(1), bucket access is O(1), chain is short (close to 1 element).
- **Worst O(n)**: if all n keys hash to the same bucket, you must walk the entire chain.

### Lookup (Get / Search)

```cpp
Value* lookup(Key k) {
    int idx = hash(k) % table_size;
    // Walk the chain at buckets[idx]
    // Compare each entry's key with k
    // Return pointer to value if found, nullptr otherwise
}
```

- **Average O(1)**: same reasoning -- short chains.
- **Worst O(n)**: entire chain must be traversed.

### Delete (Remove)

```cpp
bool remove(Key k) {
    int idx = hash(k) % table_size;
    // Walk the chain at buckets[idx]
    // Find the entry with key k
    // Remove it from the chain (linked list removal)
    // Return true if found and removed
}
```

- **Average O(1)**, **Worst O(n)**: same pattern.

### Why Worst Case Is O(n)

If the hash function is pathological (or an adversary crafts inputs), **all n keys can hash to the same bucket index**. The hash table degenerates into a single linked list:

```
Buckets:
 0 [ ]
 1 [ ]
 2 [ ]
 3 [ (k1,v1) ] → [ (k2,v2) ] → [ (k3,v3) ] → ... → [ (kn,vn) ]
 4 [ ]
 5 [ ]
```

Every operation must now scan this chain of length n. This is why the worst case is O(n), even though it almost never happens in practice with a good hash function and proper resizing.

---

## 🔹 When to Use Hash Maps / Hash Sets

This is the most important section. Hash tables are the **most frequently useful data structure** in algorithm interviews and real-world programming. If you develop an instinct for recognizing these patterns, you will solve problems faster.

### Pattern 1: Frequency Counting

**Signal**: "count occurrences", "most frequent", "top K", "majority element"

Use a hash map where keys are elements and values are counts.

```cpp
// Count character frequencies
unordered_map<char, int> freq;
for (char c : s) {
    freq[c]++;
}
// Now freq['a'] gives the count of 'a' in the string
```

**Problems that use this**: majority element, top K frequent elements, first unique character, valid anagram, sort characters by frequency.

### Pattern 2: Deduplication

**Signal**: "remove duplicates", "unique elements", "contains duplicate"

Use a hash set to track what you've already seen.

```cpp
// Check if array has any duplicates
unordered_set<int> seen;
for (int x : nums) {
    if (seen.count(x)) return true;  // duplicate found
    seen.insert(x);
}
return false;
```

**Problems that use this**: contains duplicate, longest consecutive sequence, intersection of two arrays.

### Pattern 3: Fast Lookup ("Have I Seen This Before?")

**Signal**: any time you're doing a linear scan to check existence inside a loop -- replace with a hash set.

```cpp
// BAD: O(n^2) -- linear search inside a loop
for (int x : arr1) {
    for (int y : arr2) {
        if (x == y) { /* found match */ }
    }
}

// GOOD: O(n) -- hash set lookup
unordered_set<int> set2(arr2.begin(), arr2.end());
for (int x : arr1) {
    if (set2.count(x)) { /* found match */ }
}
```

This is the **single most common optimization** you will use. Whenever you have a nested loop where the inner loop is just searching, replace it with a hash set.

### Pattern 4: Two-Sum / Complement Lookup

**Signal**: "find two elements that sum to target", "find pair with property X"

Store elements as you iterate. For each new element, check if its complement exists in the map.

```cpp
// Two Sum: find indices i, j where nums[i] + nums[j] == target
unordered_map<int, int> seen;  // value -> index
for (int i = 0; i < nums.size(); i++) {
    int complement = target - nums[i];
    if (seen.count(complement)) {
        return {seen[complement], i};
    }
    seen[nums[i]] = i;
}
```

Why this works: instead of checking every pair O(n^2), you ask the hash map "have I already seen the number I need?" in O(1).

**Problems that use this**: two sum, four sum II, count pairs with given sum.

### Pattern 5: Grouping / Categorizing

**Signal**: "group anagrams", "group by property", "partition elements"

Use a hash map where the key is the grouping criterion and the value is a list of elements.

```cpp
// Group anagrams: "eat", "tea", "ate" all share sorted key "aet"
unordered_map<string, vector<string>> groups;
for (const string& s : strs) {
    string key = s;
    sort(key.begin(), key.end());  // "eat" -> "aet"
    groups[key].push_back(s);
}
// groups["aet"] = {"eat", "tea", "ate"}
```

**Problems that use this**: group anagrams, group shifted strings, find all anagram substrings.

### Pattern 6: Caching / Memoization

**Signal**: "avoid recomputation", "overlapping subproblems", repeated function calls with same arguments

```cpp
unordered_map<int, int> memo;

int fib(int n) {
    if (n <= 1) return n;
    if (memo.count(n)) return memo[n];  // cached result
    memo[n] = fib(n - 1) + fib(n - 2);
    return memo[n];
}
```

Without memoization: O(2^n). With memoization: O(n). The hash map eliminates redundant computation.

### Pattern 7: Mapping Relationships

**Signal**: "parent-child", "graph adjacency list", "build a graph", "map one thing to another"

```cpp
// Build adjacency list for a graph
unordered_map<int, vector<int>> graph;
for (auto& edge : edges) {
    graph[edge[0]].push_back(edge[1]);
    graph[edge[1]].push_back(edge[0]);  // undirected
}

// Now graph[node] gives all neighbors of that node
```

**Problems that use this**: clone graph, course schedule, number of islands (with coordinate mapping), word ladder.

### Summary: When to Reach for a Hash Map

Ask yourself these questions:
1. Am I doing repeated lookups? --> **Hash Set/Map**
2. Am I counting things? --> **Hash Map (frequency counter)**
3. Am I checking for duplicates? --> **Hash Set**
4. Do I need to find a complement/pair? --> **Hash Map (two-sum pattern)**
5. Am I grouping items by some property? --> **Hash Map of lists**
6. Do I have nested loops where the inner one is just searching? --> **Replace inner loop with Hash Set**

If you answer **yes** to any of these, a hash table is almost certainly the right tool.

---

## 🔹 Complexity Table

| Operation        | Average Case | Worst Case | Notes                                           |
| ---------------- | ------------ | ---------- | ----------------------------------------------- |
| **Insert**       | O(1)         | O(n)       | Amortized O(1) including occasional resizing    |
| **Lookup/Get**   | O(1)         | O(n)       | Worst case when all keys collide                |
| **Delete**       | O(1)         | O(n)       | Must find the key first, then remove from chain |
| **Contains**     | O(1)         | O(n)       | Same as lookup                                  |
| **Iteration**    | O(n)         | O(n)       | Must visit all buckets + all entries             |
| **Space**        | O(n)         | O(n)       | Proportional to number of stored entries         |
| **Resize**       | O(n)         | O(n)       | Rehashes all entries, but amortized O(1) per insert |

---

## 🔹 Hash Map vs Other Data Structures

| Feature              | Hash Map             | [[Arrays\|Array]]    | Sorted Array           | Balanced BST ([[Trees]])  |
| -------------------- | -------------------- | -------------------- | ---------------------- | ------------------------- |
| **Search**           | O(1) avg             | O(n)                 | O(log n) binary search | O(log n)                  |
| **Insert**           | O(1) avg, amortized  | O(1) append / O(n)   | O(n) shift elements    | O(log n)                  |
| **Delete**           | O(1) avg             | O(n) shift           | O(n) shift elements    | O(log n)                  |
| **Ordered?**         | No                   | By index             | Yes, by value          | Yes, by key               |
| **Min/Max**          | O(n)                 | O(n)                 | O(1)                   | O(log n)                  |
| **Range query**      | Not supported        | Not supported        | O(log n + k)           | O(log n + k)              |
| **Space overhead**   | High (buckets + ptrs)| Low                  | Low                    | Moderate (node ptrs)      |
| **Cache locality**   | Poor (pointer chasing)| Excellent            | Excellent              | Moderate                  |

**When to pick what**:
- **Hash Map**: when you need fast lookup/insert/delete and don't care about order
- **Sorted Array**: when you need order and do mostly searches (few inserts)
- **Balanced BST**: when you need order AND frequent inserts/deletes (e.g., `std::map`, `TreeMap`)
- **Array**: when you access by integer index, not by key

---

## 🔹 Common Pitfalls

### 1. Mutable Keys

If you use an object as a key and then modify it, the hash changes but the entry is still stored at the OLD hash location. The key becomes "lost" -- you can't find it anymore.

```cpp
// Pseudocode showing the danger
vector<int> key = {1, 2, 3};
map[key] = "hello";

key.push_back(4);       // key is now {1, 2, 3, 4}
map[key];               // LOOKUP FAILS -- hash is different now
                        // but the old entry is still in the table, unreachable
```

**Rule**: only use immutable types (strings, integers, tuples of immutables) as hash keys.

### 2. Hash Collisions Degrading Performance

While average case is O(1), a bad hash function or adversarial inputs can cause many collisions, degrading performance toward O(n). In competitive programming, this can cause TLE (Time Limit Exceeded).

**Mitigation**: use a custom hash or a hash with randomization to prevent adversarial attacks on default hash functions.

### 3. Unordered Iteration

Hash tables do **not** maintain insertion order in most implementations (C++ `unordered_map`, Java `HashMap`). If you iterate over a hash map, the order is arbitrary and can change after rehashing.

**If you need order**: use `std::map` (C++), `TreeMap` (Java), or `LinkedHashMap` (Java, preserves insertion order).

### 4. Space Overhead

A hash table uses more memory than a plain array:
- The underlying array (buckets) is often 2x-4x the number of entries (to keep load factor low)
- Each entry in a chained implementation stores a pointer to the next node
- Each entry stores the full key (for comparison during lookup)

For large datasets where memory is critical, consider whether a sorted array with binary search might suffice.

### 5. Not Suitable for Range Queries

You cannot efficiently answer "give me all keys between 10 and 20" with a hash table. Keys are scattered across buckets with no ordering. Use a balanced BST or sorted array for range queries.

### 6. Worst-Case Guarantees

If you need guaranteed O(log n) operations (not just average), use a balanced BST instead. Hash tables only provide O(1) *average* -- the worst case is O(n).

---

## 🔹 Template Code Patterns

These are the patterns you should have memorized for interview problems. Each one is a building block that appears in dozens of problems.

### Pattern 1: Frequency Count

```cpp
// Build a frequency map of elements
unordered_map<int, int> freq;
for (int x : nums) {
    freq[x]++;
}

// Use the frequency map
for (auto& [key, count] : freq) {
    if (count > 1) {
        // key appears more than once
    }
}

// Common variant: character frequency for strings
unordered_map<char, int> charFreq;
for (char c : s) {
    charFreq[c]++;
}
```

### Pattern 2: Two-Sum (Complement Lookup)

```cpp
// Find two indices where nums[i] + nums[j] == target
unordered_map<int, int> seen;  // value -> index
for (int i = 0; i < nums.size(); i++) {
    int complement = target - nums[i];
    if (seen.count(complement)) {
        return {seen[complement], i};  // found pair
    }
    seen[nums[i]] = i;
}
return {};  // no pair found
```

**Key idea**: as you iterate, you build the map AND query it. You only need one pass.

### Pattern 3: Group By Property

```cpp
// Group strings by their sorted form (group anagrams)
unordered_map<string, vector<string>> groups;
for (const string& s : strs) {
    string key = s;
    sort(key.begin(), key.end());
    groups[key].push_back(s);
}

// Collect results
vector<vector<string>> result;
for (auto& [key, group] : groups) {
    result.push_back(group);
}
```

**Generalization**: the "key" can be any derived property -- sorted string, frequency signature, modulo class, first character, length, etc.

### Pattern 4: Check for Duplicates

```cpp
// Return true if any element appears more than once
bool containsDuplicate(vector<int>& nums) {
    unordered_set<int> seen;
    for (int x : nums) {
        if (!seen.insert(x).second) {
            return true;  // insert failed = already existed
        }
    }
    return false;
}
```

### Pattern 5: Sliding Window + Hash Map

```cpp
// Longest substring without repeating characters
int lengthOfLongestSubstring(string s) {
    unordered_map<char, int> lastSeen;  // char -> last index
    int maxLen = 0, left = 0;
    
    for (int right = 0; right < s.size(); right++) {
        char c = s[right];
        if (lastSeen.count(c) && lastSeen[c] >= left) {
            left = lastSeen[c] + 1;  // shrink window
        }
        lastSeen[c] = right;
        maxLen = max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

### Pattern 6: Prefix Sum + Hash Map

```cpp
// Count subarrays that sum to target (subarray sum equals K)
int subarraySum(vector<int>& nums, int k) {
    unordered_map<int, int> prefixCount;  // prefix_sum -> count
    prefixCount[0] = 1;  // empty prefix
    int sum = 0, count = 0;
    
    for (int x : nums) {
        sum += x;
        if (prefixCount.count(sum - k)) {
            count += prefixCount[sum - k];
        }
        prefixCount[sum]++;
    }
    return count;
}
```

**Key idea**: if `prefix[j] - prefix[i] == k`, then the subarray from `i+1` to `j` sums to `k`. The hash map lets you find matching prefix sums in O(1).

---

## 🔹 Decision Flowchart

```
Do I need to store/lookup data by a key?
│
├── YES ──► Do I need the value associated with the key?
│           │
│           ├── YES ──► Use Hash Map (unordered_map / Dictionary)
│           │
│           └── NO ───► Use Hash Set (unordered_set / HashSet)
│
└── NO ───► Do I need ordered data?
            │
            ├── YES ──► Use Sorted Array or Balanced BST
            │
            └── NO ───► Use plain Array
```

---

## 🔹 Related Concepts

- [[Collision Handling]] -- chaining vs open addressing (linear probing, quadratic probing, double hashing)
- [[Arrays]] -- the underlying structure of a hash table
- [[Singly Linked List]] -- used in chaining for collision resolution
- [[Trees]] -- balanced BSTs (red-black trees) as alternative; also used in Java's HashMap when chains get long (treeification at 8 entries)
