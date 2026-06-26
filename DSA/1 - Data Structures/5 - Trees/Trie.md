---
tags:
  - algorithms
  - data-structure
  - trie
---

## 🔹 Real-World Analogy

Think of a Trie like the autocomplete feature on your phone. As you type each letter, the system narrows down the possibilities by following the path of letters you have typed so far. The word suggestions are all the valid words that share the prefix you have entered. A Trie is literally a tree of prefixes — each path from root to a marked node represents a stored word.

## 🔹 Definition

A **Trie** (pronounced "try," from re**trie**val) is a tree-like data structure used to efficiently store and search strings, especially when dealing with **prefix-based** operations. Also called a **prefix tree** or **digital tree**.

Unlike a [[Binary Search Tree]] where each node stores a complete key and has at most 2 children, a Trie node stores a **single character** (implicitly, via its position in the parent's children) and can have up to **26 children** (for lowercase English letters) or more depending on the alphabet.

```
Words stored: "cat", "car", "card", "care", "do", "dog"

             (root)
            /      \
          [c]      [d]
          /          \
        [a]          [o]
       /   \            \
     [t*]  [r*]        [g*]
           / \
         [d*] [e*]

Nodes marked with * have isEndOfWord = true
```

Each path from root to a node with `isEndOfWord = true` spells out a stored word:
- root -> c -> a -> t = "cat"
- root -> c -> a -> r = "car"
- root -> c -> a -> r -> d = "card"
- root -> c -> a -> r -> e = "care"
- root -> d -> o = "do"
- root -> d -> o -> g = "dog"

## 🔹 Why Use a Trie?

| Operation | Hash Set | Sorted Array | Trie |
|---|---|---|---|
| Search exact word | O(m) avg | O(m log n) | O(m) |
| Prefix search ("starts with") | O(n * m) scan all | O(m log n + k) | O(m + k) |
| Autocomplete (all words with prefix) | O(n * m) | O(m log n + k) | O(m + k) |
| Lexicographic sorting | O(n log n) | Already sorted | O(n) DFS |
| Insert | O(m) avg | O(n) shift | O(m) |

Where `m` = length of the word, `n` = number of stored words, `k` = number of results.

The Trie excels at **prefix operations** — anything involving "starts with," autocomplete, or shared prefixes. A hash set can match individual lookups but cannot efficiently answer prefix queries.

## 🔹 Node Structure

Each Trie node has:
1. A map/array of **children** (one per possible character).
2. A boolean **isEndOfWord** flag indicating whether a complete word ends at this node.

```cpp
struct TrieNode {
    TrieNode* children[26];  // for lowercase English letters a-z
    bool isEndOfWord;

    TrieNode() {
        isEndOfWord = false;
        for (int i = 0; i < 26; i++)
            children[i] = nullptr;
    }
};
```

### Array vs HashMap for Children

| Approach | Pros | Cons |
|---|---|---|
| `children[26]` array | O(1) access, simple | Wastes memory if alphabet is large or sparse |
| `unordered_map<char, TrieNode*>` | Memory-efficient for sparse data | Slightly slower access, more complex |

For interview problems with lowercase English letters, the array approach is almost always preferred — simple and fast.

```cpp
// Alternative: hash map based node
struct TrieNode {
    unordered_map<char, TrieNode*> children;
    bool isEndOfWord = false;
};
```

## 🔹 Operations

### Insert

Walk through the word character by character. For each character, if the child node does not exist, create it. After the last character, mark the node as `isEndOfWord = true`.

```
Insert "car" into existing trie with "cat":

            (root)                    (root)
              |                         |
            [c]                       [c]
              |                         |
            [a]                       [a]
              |          -->         /    \
            [t*]                  [t*]   [r*]

Step 1: 'c' -- child exists, follow it
Step 2: 'a' -- child exists, follow it
Step 3: 'r' -- child does NOT exist, create it
Step 4: Mark [r] as isEndOfWord = true
```

### Search

Walk through the word character by character. If at any point the child does not exist, the word is not in the Trie. If you reach the end of the word, check `isEndOfWord` — if true, the word exists; if false, the word is only a prefix of a longer stored word but was not inserted itself.

```
Search "car":
  'c' -> exists, follow
  'a' -> exists, follow
  'r' -> exists, follow
  End of word, isEndOfWord = true --> FOUND

Search "ca":
  'c' -> exists, follow
  'a' -> exists, follow
  End of word, isEndOfWord = false --> NOT FOUND (it's a prefix, not a stored word)

Search "cab":
  'c' -> exists, follow
  'a' -> exists, follow
  'b' -> does NOT exist --> NOT FOUND
```

### StartsWith (Prefix Search)

Same as search, but you do NOT check `isEndOfWord` at the end. If you can follow all characters in the prefix without hitting a null, the prefix exists in the Trie.

```
StartsWith "ca":
  'c' -> exists, follow
  'a' -> exists, follow
  Reached end of prefix without hitting null --> PREFIX EXISTS
```

## 🔹 Template Code (Complete Trie Class)

```cpp
class Trie {
private:
    struct TrieNode {
        TrieNode* children[26];
        bool isEndOfWord;

        TrieNode() : isEndOfWord(false) {
            memset(children, 0, sizeof(children));
        }
    };

    TrieNode* root;

public:
    Trie() {
        root = new TrieNode();
    }

    // Insert a word into the trie
    // Time: O(m), Space: O(m) worst case (all new nodes)
    void insert(const string& word) {
        TrieNode* curr = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!curr->children[idx])
                curr->children[idx] = new TrieNode();
            curr = curr->children[idx];
        }
        curr->isEndOfWord = true;
    }

    // Search for an exact word
    // Time: O(m), Space: O(1)
    bool search(const string& word) {
        TrieNode* node = find(word);
        return node != nullptr && node->isEndOfWord;
    }

    // Check if any word starts with the given prefix
    // Time: O(m), Space: O(1)
    bool startsWith(const string& prefix) {
        return find(prefix) != nullptr;
    }

private:
    // Helper: traverse the trie following the given string
    // Returns the node at the end, or nullptr if path doesn't exist
    TrieNode* find(const string& s) {
        TrieNode* curr = root;
        for (char c : s) {
            int idx = c - 'a';
            if (!curr->children[idx])
                return nullptr;
            curr = curr->children[idx];
        }
        return curr;
    }
};
```

### Usage Example

```cpp
Trie trie;
trie.insert("apple");
trie.insert("app");
trie.insert("application");

trie.search("app");          // true  (exact word inserted)
trie.search("ap");           // false (prefix only, not inserted)
trie.search("apple");        // true
trie.startsWith("app");      // true  (prefix exists)
trie.startsWith("apl");      // false (no word starts with "apl")
```

## 🔹 Collecting All Words with a Given Prefix

A common extension: given a prefix, return all stored words that start with it. This is the core of autocomplete.

```cpp
void collectWords(TrieNode* node, string& current, vector<string>& results) {
    if (!node) return;

    if (node->isEndOfWord)
        results.push_back(current);

    for (int i = 0; i < 26; i++) {
        if (node->children[i]) {
            current.push_back('a' + i);
            collectWords(node->children[i], current, results);
            current.pop_back();  // backtrack
        }
    }
}

vector<string> autocomplete(Trie& trie, const string& prefix) {
    TrieNode* node = trie.find(prefix);  // navigate to end of prefix
    if (!node) return {};

    vector<string> results;
    string current = prefix;
    collectWords(node, current, results);
    return results;
}
```

```
Trie contains: "car", "card", "care", "cat"
autocomplete("car") --> ["car", "card", "care"]
autocomplete("ca")  --> ["car", "card", "care", "cat"]
autocomplete("d")   --> []  (no words start with "d")
```

## 🔹 ASCII Visualization — Building a Trie Step by Step

```
Insert "to":
    (root)
       |
     [t]
       |
     [o*]

Insert "tea":
    (root)
       |
     [t]
     / \
   [o*] [e]
          |
         [a*]

Insert "ten":
    (root)
       |
     [t]
     / \
   [o*] [e]
        / \
      [a*] [n*]

Insert "inn":
      (root)
      /    \
    [i]    [t]
     |     / \
    [n]  [o*] [e]
     |        / \
    [n*]    [a*] [n*]

Insert "in":
      (root)
      /    \
    [i]    [t]
     |     / \
    [n*]  [o*] [e]
     |        / \
    [n*]    [a*] [n*]

Note: "in" marks [n] at depth 2 as isEndOfWord,
even though [n] at depth 3 was already marked for "inn".
Both "in" and "inn" coexist because isEndOfWord is per-node.
```

## 🔹 Complexity Summary

| Operation | Time | Space |
|---|---|---|
| Insert | O(m) | O(m) worst case per word |
| Search | O(m) | O(1) |
| StartsWith | O(m) | O(1) |
| Autocomplete (all matches) | O(m + k) | O(longest word) recursion |
| Total space for n words | -- | O(n * m * ALPHABET_SIZE) worst case |

Where `m` = length of the word/prefix, `k` = total characters in all matching results, `ALPHABET_SIZE` = 26 for lowercase English.

**Space note:** In the worst case (no shared prefixes), a Trie can use significant memory because each node has an array of 26 pointers. In practice, shared prefixes make Tries much more space-efficient than storing each word independently. For very large dictionaries, consider a compressed trie (Patricia tree / Radix tree) that merges single-child chains.

## 🔹 Use Cases

1. **Autocomplete / Search Suggestions** — As the user types, find all words matching the current prefix. Used in search engines, IDEs, and mobile keyboards.

2. **Spell Checker** — Check if a word exists in a dictionary. Suggest corrections by exploring nearby paths in the Trie (edit distance 1 or 2).

3. **Prefix Matching / Filtering** — IP routing tables (longest prefix match), phone number lookup, filtering log entries by prefix.

4. **Word Games** — Boggle, Scrabble solvers — efficiently check if a sequence of letters forms a valid word or prefix.

5. **Word Search (LeetCode 212)** — Search for multiple words simultaneously in a 2D grid. A Trie lets you prune invalid paths early.

6. **Counting Distinct Prefixes** — Count how many words share each prefix. Add a `count` field to each node, incremented during insertion.

7. **Lexicographic Sorting** — A DFS traversal of a Trie visits all stored words in alphabetical (lexicographic) order for free.

## 🔹 Trie vs Other Data Structures

| Scenario | Best Choice | Why |
|---|---|---|
| Exact word lookup only | Hash Set | O(1) average, simpler |
| Prefix queries needed | Trie | O(m) prefix search, autocomplete |
| Sorted order needed | Trie or Balanced BST | Trie DFS gives sorted order |
| Memory constrained | Hash Set | Trie nodes are memory-heavy |
| Pattern matching (regex) | Trie (limited) or Suffix Tree | Tries handle prefix patterns |

## 🔹 Common Pitfalls

1. **Confusing search and startsWith.** `search("app")` returns true only if "app" was explicitly inserted. `startsWith("app")` returns true if ANY word starting with "app" exists (e.g., "apple"). The only difference is checking `isEndOfWord`.

2. **Forgetting that "in" and "inn" can coexist.** The node for 'n' (after 'i') can be marked as `isEndOfWord = true` for "in" while still having a child 'n' leading to "inn". Each node independently tracks whether a word ends there.

3. **Memory overhead with array-based children.** Each node allocates space for 26 pointers (208 bytes on 64-bit). For large tries with few shared prefixes, this adds up. Consider hash-map based nodes or compressed tries if memory is a concern.

4. **Not handling deletion properly.** Deleting a word from a Trie means setting `isEndOfWord = false`, but you should also clean up nodes that are no longer part of any word (no children and not end-of-word). Deletion is rarely asked in interviews but good to be aware of.

5. **Case sensitivity.** The standard `c - 'a'` indexing only works for lowercase letters. If the problem includes uppercase, digits, or other characters, adjust the alphabet size and indexing accordingly.

## 🔹 Delete Operation (For Completeness)

```cpp
// Returns true if the node should be deleted (has no children and is not end of another word)
bool deleteHelper(TrieNode* node, const string& word, int depth) {
    if (!node) return false;

    if (depth == word.size()) {
        if (!node->isEndOfWord) return false;  // word not found

        node->isEndOfWord = false;

        // If node has no children, it can be deleted
        for (int i = 0; i < 26; i++)
            if (node->children[i]) return false;
        return true;
    }

    int idx = word[depth] - 'a';
    if (deleteHelper(node->children[idx], word, depth + 1)) {
        delete node->children[idx];
        node->children[idx] = nullptr;

        // Check if current node can also be deleted
        if (!node->isEndOfWord) {
            for (int i = 0; i < 26; i++)
                if (node->children[i]) return false;
            return true;
        }
    }

    return false;
}

void deleteWord(const string& word) {
    deleteHelper(root, word, 0);
}
```
