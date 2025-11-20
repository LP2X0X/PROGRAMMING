---
tags: 
 - regex
 - metacharacter
---

Probably the easiest metacharacters to understand are !^" (car et) and !$" (dollar), which represent the start and end, respectively, of the line of text as it is being checked.

| Symbol | Meaning                          | Matches example                          | Does **not** match                  |
|--------|----------------------------------|------------------------------------------|-------------------------------------|
| `^`    | **Start of string** (or start of line with `m` flag) | `^abc` → "abcde"  <br> `^abc` → "xyzabc" (no) | "xyzabc" (doesn't start with abc) |
| `$`    | **End of string** (or end of line with `m` flag)   | `abc$` → "xyzabc" <br> `abc$` → "abcde" (no)  | "abcde" (doesn't end with abc)    |

```ad-note
Note that this is different than the hat used inside a set of bracket _[^...]_ for excluding characters, which can be confusing when reading regular expressions.
```

### Combined
```regex
^abc$   → matches only the exact string "abc" (nothing before or after)
```

### With multiline flag (`m` in JS/Python/PCRE)
- `^` = start of a **line**
- `$` = end of a **line** (including before newline)

Example (`m` flag on):
```regex
^hello.*$
```
→ matches every line that begins with "hello" and goes to its end.

### Quick rules
- No `m` flag → `^` and `$` refer to the **entire string**.
- With `m` flag → `^` and `$` refer to **each line**.
- Always escape if you want literal `^` or `$`: `\^` or `\$`.

That’s it! Use `^...$` when you want **exact/full-string** match.