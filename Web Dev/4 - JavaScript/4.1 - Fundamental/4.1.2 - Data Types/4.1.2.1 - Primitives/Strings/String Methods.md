---
tags:
  - js
  - string
  - methods
  - fundamental
---

Here's a **list of frequently used `String` methods** in JavaScript, grouped by purpose, with examples:

---

## 🔤 **1. Checking & Searching**

| Method          | Description                            | Example                           |
| --------------- | -------------------------------------- | --------------------------------- |
| `includes()`    | Checks if substring exists             | `"hello".includes("ell") → true`  |
| `startsWith()`  | Checks if string starts with substring | `"hello".startsWith("he") → true` |
| `endsWith()`    | Checks if string ends with substring   | `"hello".endsWith("lo") → true`   |
| `indexOf()`     | First index of substring (or -1)       | `"hello".indexOf("l") → 2`        |
| `lastIndexOf()` | Last index of substring                | `"hello".lastIndexOf("l") → 3`    |
| `match()`       | Regex match (returns array or null)    | `"abc123".match(/\d+/) → ['123']` |
| `search()`      | Regex search (returns index or -1)     | `"abc123".search(/\d+/) → 3`      |

---

## ✂️ **2. Extracting & Slicing**

| Method                  | Description                    | Example                           |
| ----------------------- | ------------------------------ | --------------------------------- |
| `slice(start, end)`     | Extracts part of string        | `"hello".slice(1, 4) → "ell"`     |
| `substring(start, end)` | Similar to slice (no negative) | `"hello".substring(1, 4) → "ell"` |
| `substr(start, length)` | Deprecated                     | `"hello".substr(1, 3) → "ell"`    |
| `charAt(index)`         | Returns character at index     | `"hello".charAt(1) → "e"`         |
| `charCodeAt(index)`     | Unicode code at index          | `"A".charCodeAt(0) → 65`          |

---

## 🔁 **3. Modifying Strings**

| Method                      | Description                                                 | Example                                |
| --------------------------- | ----------------------------------------------------------- | -------------------------------------- |
| `toUpperCase()`             | Converts to uppercase                                       | `"hello".toUpperCase() → "HELLO"`      |
| `toLowerCase()`             | Converts to lowercase                                       | `"HELLO".toLowerCase() → "hello"`      |
| `replace()`                 | Replace first match or regex                                | `"aabb".replace("a", "z") → "zabb"`    |
| `replaceAll()`              | Replace all matches                                         | `"aabb".replaceAll("a", "z") → "zzbb"` |
| `trim()`                    | Removes anything that counts as "whitespace" from both ends | `" hi ".trim() → "hi"`                 |
| `trimStart()` / `trimEnd()` | Leading/trailing trim                                       | `" hi ".trimStart() → "hi "`           |
| `repeat(n)`                 | Repeats string n times                                      | `"ab".repeat(3) → "ababab"`            |
| `padStart(len, str)`        | Pads at start                                               | `"5".padStart(3, "0") → "005"`         |
| `padEnd(len, str)`          | Pads at end                                                 | `"5".padEnd(3, "0") → "500"`           |

---

## 🧩 **4. Splitting & Joining**

| Method             | Description               | Example                              |
| ------------------ | ------------------------- | ------------------------------------ |
| `split(separator)` | Converts string to array  | `"a,b,c".split(",") → ['a','b','c']` |
| `join(separator)`  | (From array) makes string | `['a','b'].join("-") → "a-b"`        |

---

## ✅ **5. Others**

| Method            | Description                     | Example                        |
| ----------------- | ------------------------------- | ------------------------------ |
| `localeCompare()` | Compares strings (locale-aware) | `"a".localeCompare("b") → -1`  |
| `normalize()`     | Unicode normalization           | Used with accents, emoji, etc. |

---

### ⚠️ Notes:

- Many of these methods **return new strings** (they do **not modify** the original).
    
- Methods like `match`, `replace`, `search` are powerful when used with **regular expressions**.
    
