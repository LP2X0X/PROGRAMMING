---
tags:
  - algorithms
  - data-structure
  - string
---

## 🔹 Intuition

A string is a **sequence of characters** -- think of it as a necklace of beads where each bead is a letter, digit, or symbol. Just like you can inspect any bead by its position, reverse the necklace, or compare two necklaces bead-by-bead, strings support the same operations on characters.

Under the hood, a string is just an **array of characters** with some extra conventions. Everything you know about [[Arrays]] applies here -- contiguous memory, O(1) index access, O(n) insertion/deletion. The twist is that strings carry additional semantics: encoding, termination, and in many languages, **immutability**.

---

## 🔹 String Representation in Memory

There are two fundamental ways languages store strings:

### Null-Terminated (C-Style)

The string ends with a special `'\0'` (null) character. The length must be computed by scanning.

```
char s[] = "HELLO";
```

```
Index:    0     1     2     3     4     5
        +-----+-----+-----+-----+-----+------+
        | 'H' | 'E' | 'L' | 'L' | 'O' | '\0' |
        +-----+-----+-----+-----+-----+------+
Address: 0x00  0x01  0x02  0x03  0x04  0x05

         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^   ^^^^^^
              visible characters       terminator
```

- `strlen()` is **O(n)** -- must scan to find `'\0'`.
- Buffer must hold `length + 1` characters.
- Forgetting the null terminator causes undefined behavior (buffer overruns, reading garbage).

### Length-Prefixed

The string stores its length explicitly. Used by C++ `std::string`, Java `String`, C# `string`, Python `str`, etc.

```
struct String {
    int length;    // stored explicitly
    char* data;    // pointer to character buffer
};
```

```
         +--------+-----------------------------------+
         | len: 5 | 'H' | 'E' | 'L' | 'L' | 'O'     |
         +--------+-----------------------------------+
```

- `length` is **O(1)** -- just read the stored field.
- Can contain null bytes within the string.
- No ambiguity about where the string ends.

---

## 🔹 Immutability

Most modern languages (Java, C#, Python, JavaScript, Go) make strings **immutable** -- once created, the character data cannot be changed in place.

**Why immutability?**

- **Thread safety** -- no locks needed for concurrent reads.
- **Hashing** -- hash can be computed once and cached (strings are common hash map keys).
- **Security** -- prevents accidental or malicious modification of string data passed between functions.
- **String interning** -- identical strings can share the same memory.

**The performance trap:**

```cpp
// This looks innocent but is O(n^2) for immutable strings
string result = "";
for (int i = 0; i < n; i++) {
    result = result + items[i];  // creates a NEW string each iteration
}
```

Each `+` allocates a new string and copies all previous characters:

```
Iteration 1: copy 1 char           -> 1 copy
Iteration 2: copy 2 chars          -> 2 copies
Iteration 3: copy 3 chars          -> 3 copies
...
Iteration n: copy n chars          -> n copies
                                   ─────────
Total copies: 1 + 2 + ... + n = n(n+1)/2 = O(n^2)
```

In C/C++, strings (char arrays / `std::string`) **are mutable**, so you can modify characters in place. But the concatenation trap still applies if you keep creating new strings.

---

## 🔹 StringBuilder Pattern

The fix for the O(n^2) concatenation problem: use a **resizable buffer** that appends in amortized O(1).

```
StringBuilder / StringBuffer / std::ostringstream
```

**Without StringBuilder (O(n^2)):**

```
Append "AB":   [ A | B ]                           -> copy 2
Append "CD":   [ A | B | C | D ]                   -> copy 4 (new alloc)
Append "EF":   [ A | B | C | D | E | F ]           -> copy 6 (new alloc)
               ^^^ entire string copied each time
```

**With StringBuilder (amortized O(n)):**

```
Buffer (capacity 8):

Append "AB":   [ A | B | _ | _ | _ | _ | _ | _ ]   -> write 2, no copy
                       ^cursor
Append "CD":   [ A | B | C | D | _ | _ | _ | _ ]   -> write 2, no copy
                               ^cursor
Append "EF":   [ A | B | C | D | E | F | _ | _ ]   -> write 2, no copy
                                       ^cursor

Final .toString():  "ABCDEF"  -> one allocation at the end
```

The buffer doubles in capacity when full (amortized O(1) append). Total work: O(n) instead of O(n^2).

**Rule of thumb:** If you are concatenating strings in a loop, use a StringBuilder (or equivalent).

---

## 🔹 ASCII and Unicode Basics

### ASCII (7-bit, 128 characters)

Key ranges to memorize for algorithm problems:

| Range     | Characters | Decimal  | Notes                        |
| --------- | ---------- | -------- | ---------------------------- |
| `'0'-'9'` | Digits     | 48 - 57  | `c - '0'` gives numeric val |
| `'A'-'Z'` | Uppercase  | 65 - 90  | `c - 'A'` gives 0-25 index  |
| `'a'-'z'` | Lowercase  | 97 - 122 | `c - 'a'` gives 0-25 index  |
| `' '`     | Space      | 32       |                              |

Useful conversions:

```cpp
// char to index (for frequency arrays)
int idx = c - 'a';       // 'a'->0, 'b'->1, ..., 'z'->25

// uppercase <-> lowercase (differ by 32)
char lower = upper + 32;       // 'A' + 32 = 'a'
char upper = lower - 32;       // 'a' - 32 = 'A'
// Or using bitwise:
char lower = upper | 0x20;     // set bit 5
char upper = lower & ~0x20;    // clear bit 5

// check if alphabetic
bool isAlpha = (c >= 'A' && c <= 'Z') || (c >= 'a' && c <= 'z');

// check if digit
bool isDigit = c >= '0' && c <= '9';
```

### Unicode and UTF-8

- **Unicode** assigns a code point (number) to every character in every language.
- **UTF-8** is a variable-width encoding:

| Bytes | Code Point Range  | Example           |
| ----- | ----------------- | ----------------- |
| 1     | U+0000 - U+007F   | ASCII (A, 1, !)   |
| 2     | U+0080 - U+07FF   | Latin, Greek, etc |
| 3     | U+0800 - U+FFFF   | CJK, most scripts |
| 4     | U+10000 - U+10FFFF | Emoji, rare chars |

**For algorithm problems:** Unless stated otherwise, assume ASCII-only strings. If the problem says "lowercase English letters", you get exactly 26 characters and can use `int freq[26]`.

---

## 🔹 Operations and Time Complexity

| Operation                      | Time Complexity              | Notes                                         |
| ------------------------------ | ---------------------------- | --------------------------------------------- |
| Access `s[i]`                  | O(1)                         | Direct array index                            |
| Length                         | O(1) length-prefixed         | O(n) for C-style `strlen`                     |
| Concatenate `s1 + s2`         | O(n + m)                     | Must allocate and copy both                   |
| Substring `s.substr(i, k)`    | O(k)                         | Copy k characters                             |
| Search (find substring)       | O(n * m) naive               | KMP achieves O(n + m)                         |
| Compare `s1 == s2`            | O(min(n, m))                 | Character-by-character                        |
| Reverse                        | O(n)                         | Swap from both ends                           |
| Insert at position i           | O(n)                         | Shift characters right                        |
| Delete at position i           | O(n)                         | Shift characters left                         |
| Append (StringBuilder)         | O(1) amortized               | O(n) worst case on resize                     |
| Check prefix/suffix            | O(k)                         | k = length of prefix/suffix                   |

---

## 🔹 Common Techniques for Algorithm Problems

### 1. Two-Pointer Technique

Use two pointers (often `left` and `right`) moving toward each other or in the same direction. Perfect for palindromes, reversals, and partitioning.

See: [[Two Pointers]]

```
   left                    right
    |                        |
    v                        v
   [R] [A] [C] [E] [C] [A] [R]

    Compare s[left] vs s[right]
    If equal, move both inward
    If not equal, NOT a palindrome
```

### 2. Sliding Window

Maintain a window `[left, right]` over the string. Expand `right` to include more, shrink `left` to exclude. Track window state with a hash map or frequency array.

See: [[Sliding Window]]

```
    "ADOBECODEBANC"     target = "ABC"

    left    right
     |        |
     v        v
    [A D O B E C] O D E B A N C     -> window contains A,B,C -- valid!
       |        |
       v        v
     A [D O B E C O] D E B A N C   -> shrink from left, check...
```

Use for: longest substring without repeating chars, minimum window substring, longest substring with at most K distinct characters.

### 3. Character Frequency Counting

The workhorse technique for anagram, permutation, and character-based problems.

**Option A -- Fixed-size array (when charset is known):**

```cpp
int freq[26] = {0};                 // for lowercase English letters
for (char c : s) {
    freq[c - 'a']++;
}
```

**Option B -- Hash map (when charset is large or unknown):**

```cpp
unordered_map<char, int> freq;
for (char c : s) {
    freq[c]++;
}
```

Use the array approach when the problem says "lowercase English letters" -- it is faster (no hashing overhead) and uses less memory.

See: [[Hash Map and Hash Set]]

### 4. Anagram Detection

Two strings are anagrams if they have the same character frequencies. Two approaches:

```
"listen" -> freq: {l:1, i:1, s:1, t:1, e:1, n:1}
"silent" -> freq: {s:1, i:1, l:1, e:1, n:1, t:1}
             Same frequencies -> anagram!
```

- **Sort both and compare:** O(n log n) time, O(1) extra space.
- **Frequency count and compare:** O(n) time, O(1) extra space (fixed 26-char array).

### 5. Palindrome Check

A string reads the same forward and backward. Use two pointers converging from both ends.

```
   "racecar"

    L              R
    |              |
    r  a  c  e  c  a  r
    r == r  ->  move inward
       a == a  ->  move inward
          c == c  ->  move inward
             e  ->  L >= R, done. PALINDROME.
```

---

## 🔹 Template Code

### Palindrome Check

```cpp
bool isPalindrome(const string& s) {
    int left = 0, right = s.size() - 1;
    while (left < right) {
        if (s[left] != s[right]) return false;
        left++;
        right--;
    }
    return true;
}

// Variant: ignore non-alphanumeric, case-insensitive
bool isPalindromeClean(const string& s) {
    int left = 0, right = s.size() - 1;
    while (left < right) {
        while (left < right && !isalnum(s[left]))  left++;
        while (left < right && !isalnum(s[right])) right--;
        if (tolower(s[left]) != tolower(s[right])) return false;
        left++;
        right--;
    }
    return true;
}
```

See: [[Valid Palindrome]]

### Anagram Detection

```cpp
bool isAnagram(const string& s, const string& t) {
    if (s.size() != t.size()) return false;

    int freq[26] = {0};
    for (int i = 0; i < s.size(); i++) {
        freq[s[i] - 'a']++;
        freq[t[i] - 'a']--;
    }
    for (int i = 0; i < 26; i++) {
        if (freq[i] != 0) return false;
    }
    return true;
}
```

See: [[Valid Anagram]]

### Character Frequency Count

```cpp
// Count frequency of each character in a string
// Returns array where index 0 = 'a', 1 = 'b', ..., 25 = 'z'
void charFrequency(const string& s, int freq[26]) {
    memset(freq, 0, 26 * sizeof(int));
    for (char c : s) {
        freq[c - 'a']++;
    }
}

// Find the first non-repeating character
int firstUnique(const string& s) {
    int freq[26] = {0};
    for (char c : s) freq[c - 'a']++;
    for (int i = 0; i < s.size(); i++) {
        if (freq[s[i] - 'a'] == 1) return i;
    }
    return -1;  // all characters repeat
}
```

### Sliding Window -- Longest Substring Without Repeating Characters

```cpp
int longestUniqueSubstring(const string& s) {
    int freq[128] = {0};          // ASCII character set
    int left = 0, maxLen = 0;

    for (int right = 0; right < s.size(); right++) {
        freq[s[right]]++;

        // Shrink window while we have a duplicate
        while (freq[s[right]] > 1) {
            freq[s[left]]--;
            left++;
        }
        maxLen = max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

---

## 🔹 Common Pitfalls

**1. Off-by-one with null terminator (C-style)**

```cpp
char buf[5];
strcpy(buf, "HELLO");  // BUG: "HELLO" is 6 bytes (5 chars + '\0')
                        // Buffer overflow!
// Fix: char buf[6]; or better, char buf[strlen("HELLO") + 1];
```

**2. Immutability surprise in concatenation loops**

```cpp
// Java / C# / Python -- this is O(n^2), not O(n)
string result = "";
for (int i = 0; i < 100000; i++) {
    result += "x";   // new allocation + full copy each time
}
// Fix: use StringBuilder (Java/C#) or list + join (Python)
```

**3. Encoding issues with `s.length()` vs character count**

```
// UTF-8: a multi-byte character counts as multiple bytes
"café"  -- 'e' + combining accent = 5 code points but 4 visible chars
s.length()    -- returns code unit count, NOT visible character count
```

For algorithm problems, this rarely matters (problems use ASCII), but in production code, always be aware of encoding.

**4. Comparing strings with `==` vs `.equals()`**

In Java, `==` compares references (memory addresses), not content. Use `.equals()` for content comparison. In C++, `==` on `std::string` does compare content correctly.

**5. Forgetting empty string edge case**

```cpp
// This crashes on empty string
char first = s[0];  // undefined behavior if s is empty

// Fix: always check
if (!s.empty()) {
    char first = s[0];
}
```

---

## 🔹 Key Takeaways

1. Strings are character arrays with extra conventions (termination, encoding, immutability).
2. Most string operations are **O(n)** -- there is no free lunch with sequential data.
3. Use **StringBuilder** for concatenation in loops -- turns O(n^2) into O(n).
4. Use **`int freq[26]`** for lowercase letter problems -- faster than hash maps.
5. Master **two-pointer** and **sliding window** -- they solve most string interview problems.
6. Always consider edge cases: empty string, single character, all same characters.

---

## 🔹 Related Concepts

- [[Arrays]] -- strings are specialized arrays; all array techniques apply
- [[Hash Map and Hash Set]] -- frequency counting, anagram detection
- [[Stack]] -- matching parentheses, expression parsing
- [[Two Pointers]] -- palindrome checks, string reversal
- [[Sliding Window]] -- substring problems, window-based frequency tracking
- [[Valid Palindrome]] -- classic two-pointer string problem
- [[Valid Anagram]] -- character frequency comparison
- [[Two Sum]] -- related hash-map lookup pattern
- [[Big O - Definition]] -- complexity analysis fundamentals
- [[Sorting]] -- sorted string comparison for anagram detection
