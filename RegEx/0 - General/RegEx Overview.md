---
tags: 
 - regex
 - general
 - overview
---

### Regex (Regular Expressions) – General Overview

Regular expressions (regex or regexp) are a powerful tool for searching, matching, validating, and manipulating text using patterns. They are supported in almost every programming language and tool (Python, JavaScript, Java, Perl, grep, sed, awk, etc.).

---

## 🔍 What is Regex?

A regular expression is a formulaic way of describing a set of strings that follow a certain pattern. It is used to:

- **Search** for a specific pattern within a string (e.g., finding all email addresses in a document).
    
- **Validate** input to ensure it conforms to a certain structure (e.g., checking if a password meets complexity requirements).
    
- **Replace** occurrences of a pattern with another string (e.g., changing all instances of a date format).
    
- **Split** a string into an array/list based on a pattern as a delimiter.
    

### Key Components

A regex pattern is constructed using two main types of characters:

1. **Literal Characters:** These match themselves exactly (e.g., `a`, `1`, `z`).
    
2. **Metacharacters:** These have special meanings and define the pattern logic (e.g., `.` for any single character, `*` for zero or more repetitions).
    

---

## 🍦 Regex "Flavors" (Implementations)

While the core concept of regular expressions is universal, the exact set of features and syntax can vary slightly depending on the programming language or tool being used. These variations are often referred to as **regex flavors**.

The two most widely recognized families of regex are:

### 1. POSIX (Portable Operating System Interface)

This is an older, more formalized standard, often found in Unix tools like `grep` and `sed`. It has two main sub-flavors:

- **Basic Regular Expressions (BRE):** Requires certain metacharacters (like `{}`, `()`) to be escaped with a backslash to be treated specially.
    
- **Extended Regular Expressions (ERE):** Treats metacharacters as special characters by default, requiring fewer backslashes, making the pattern generally easier to read.
    

### 2. PCRE (Perl Compatible Regular Expressions)

This is the most common and powerful flavor, originating from the **Perl** language. It has been adopted by many modern languages and tools, including:

- **PHP**
    
- **Python** (using the `re` module)
    
- **Java**
    
- **JavaScript**
    
- **C#/.NET**
    

PCRE is known for its extensive set of features, such as **non-capturing groups**, **lookaheads** and **lookbehinds**, and **possessive quantifiers**, which often make it much more expressive and efficient than the POSIX standard.

--- 

#### Basic Concepts

| Symbol/Category       | Meaning                                              | Example                  | Matches                              |
|-----------------------|------------------------------------------------------|--------------------------|--------------------------------------|
| `.`                   | Any character except newline                         | `a.c`                    | "abc", "axc", "a c"                  |
| `^`                   | Start of string/line                                 | `^Hello`                 | "Hello world" (but not "Say Hello")  |
| `$`                   | End of string/line                                   | `world$`                 | "Hello world"                        |
| `*`                   | 0 or more of previous                                | `a*`                     | "", "a", "aaa"                       |
| `+`                   | 1 or more of previous                                | `a+`                     | "a", "aaa" (not empty)               |
| `?`                   | 0 or 1 of previous (makes it optional)              | `a?`                     | "", "a"                              |
| `{n}`                 | Exactly n times                                      | `a{3}`                   | "aaa"                                |
| `{n,}`                | n or more times                                      | `a{2,}`                  | "aa", "aaa", etc.                    |
| `{n,m}`               | Between n and m times                                | `a{2,4}`                 | "aa", "aaa", "aaaa"                  |
| `|`                   | OR (alternation)                                     | `cat|dog`                | "cat" or "dog"                       |
| `( )`                 | Capturing group                                      | `(ab)+`                  | "ab", "abab" – group "ab" repeated   |
| `(?: )`               | Non-capturing group                                  | `(?:ab)+`                | same as above but doesn’t capture    |
| `[abc]`               | Character class – any one inside                     | `[aeiou]`                | any vowel                            |
| `[^abc]`              | Negated class – anything except inside               | `[^0-9]`                 | non-digit                            |
| `[a-z]`               | Range                                                | `[A-Z0-9]`               | uppercase letters or digits          |
| `\d`                  | Digit (same as [0-9])                                | `\d{3}`                  | "123"                                |
| `\D`                  | Non-digit                                            |                          |                                      |
| `\w`                  | Word character [a-zA-Z0-9_]                          |                          |                                      |
| `\W`                  | Non-word character                                   |                          |                                      |
| `\s`                  | Whitespace (space, tab, newline)                     |                          |                                      |
| `\S`                  | Non-whitespace                                       |                          |                                      |
| `\b`                  | Word boundary                                        | `\bcat\b`                | "cat" but not "catch"                |
| `\`                   | Escape special characters                            | `\.`                     | literal dot                          |

#### Common Regex Elements

| **Element**              | **Description**                                       | **Example Pattern**    | **Matches**                                                                           |
| ------------------------ | ----------------------------------------------------- | ---------------------- | ------------------------------------------------------------------------------------- |
| **Anchors**              | Match positions, not characters.                      | `^`, `$`               | Beginning (`^`) or end (`$`) of a string/line.                                        |
| **Quantifiers**          | Specify how many times a preceding element can occur. | `*`, `+`, `?`, `{n,m}` | Zero or more (`*`), one or more (`+`), zero or one (`?`), $n$ to $m$ times (`{n,m}`). |
| **Character Classes**    | Match any one of a set of characters.                 | `[abc]`, `[0-9]`       | `a`, `b`, or `c`; any digit from 0 to 9.                                              |
| **Shorthands**           | Convenient shortcuts for common character classes.    | `\d`, `\w`, `\s`       | Any digit, word character (alphanumeric + underscore), or whitespace character.       |
| **Grouping & Capturing** | Group elements together and capture the matched text. | `(abc)`                | Matches the sequence "abc" and saves it.                                              |

#### Common Useful Patterns (Real-world examples)

| Purpose                  | Regex Pattern                          | Explanation                                      |
|--------------------------|----------------------------------------|--------------------------------------------------|
| Email (basic)            | `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}` | Simple but catches most emails                   |
| Email (strict)           | `^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$` | RFC-compliant-ish                              |
| US phone number          | `\b\(?(\d{3})\)?[-. ]?(\d{3})[-. ]?(\d{4})\b` | (555) 123-4567, 555-123-4567, etc.             |
| Date YYYY-MM-DD          | `\d{4}-\d{2}-\d{2}`                    | 2025-11-19                                       |
| Date MM/DD/YYYY          | `\b(0[1-9]|1[0-2])/(0[1-9]|[12]\d|3[01])/\d{4}\b` | Validates month/day range roughly              |
| Hex color                | `^#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3})$`  | #fff or #ffffff                                  |
| IPv4 address             | `^(?:(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(?:25[0-5]|2[0-4]\d|[01]?\d\d?)$` | 192.168.1.1                                  |
| Password (8+ chars, upper, lower, digit, special) | `^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$` | Strong password rule                             |
| Extract URLs (basic)     | `https?://[^\s]+`                      | Very simple URL matcher                          |

#### Flags / Modifiers (depend on engine)
- `i` → case-insensitive
- `g` → global (find all matches)
- `m` → multiline (^ and $ match start/end of lines)
- `s` → dotall (`.` matches newline too)
- `u` → Unicode support (in JS/Python)

#### Important Notes
- Different regex engines have slight differences:
  - JavaScript (ECMAScript) – limited lookbehind
  - Python (re module) – very standard
  - PCRE (Perl, PHP, many tools) – most features
  - .NET – extremely powerful
- Always test your regex! Great tools:
  - https://regex101.com (best, shows explanation)
  - https://regextester.com
  - https://regexr.com

#### Quick cheat sheet (most used)
```
.   any char
^   start
$   end
*   0 or more
+   1 or more
?   0 or 1
\d  digit
\w  word char
\s  whitespace
[]  character class
()  capture group
(?:) non-capturing
|   or
```
