---
tags: 
 - regex
 - term
---

A **character class** is a regex feature written inside **square brackets `[ ]`** that matches **exactly one character** from a _set_ of allowed characters.

Think of it as:

> “Match _any one_ of these characters.”

Example:

```
[abc]
```

Matches **a** or **b** or **c**.

---

# 📌 **Why character classes exist**

They let you match multiple possible characters **without writing many ORs** like:

```
a|b|c
```

Instead:

```
[abc]
```

Cleaner, faster, easier.

---

# 🧩 **Common Types of Character Classes**

## 1. **Simple list**

```
[xyz]
```

Matches **x OR y OR z**.

## 2. **Ranges**

Within a character class, the character-class metacharacter '-' (dash) indicates a range of characters:

```
[a-z]      // all lowercase letters
[A-Z]      // all uppercase letters
[0-9]      // all digits
```

Ranges save a LOT of typing.

---

## 3. **Combining ranges**

```
[a-zA-Z0-9]
```

Matches any **alphanumeric** character.

---

## 4. **Negated Character Class**

```
[^a-z]
```

Matches any character **except** lowercase letters.

`^` at the _first position_ means **NOT**.

---

## 5. **Special built-in classes inside [ ]**

### `\d` → digits

```
[\d]   same as [0-9]
```

### `\w` → word characters

```
[\w]   same as [a-zA-Z0-9_]
```

### `\s` → whitespace

```
[\s]   space, tab, newline
```

They work inside classes.

---

## 6. **Escaping inside character classes**

Inside `[ ]`, only a few characters need escaping:

- `]`
    
- `-`
    
- `^` (only if at the beginning)
    

Example:

```
[-a-z]   // dash allowed as first char or escaped
```

---

# ⭐ Practical Examples

### **Match a vowel**

```
[aeiou]
```

### **Match a hex digit**

```
[0-9A-Fa-f]
```

### **Match anything except numbers**

```
[^0-9]
```

### **Match one symbol**

```
[!@#$%^&*]
```

### **Match a letter or underscore**

```
[a-zA-Z_]
```

---

# ✔ Summary (easy to remember)

|Syntax|Meaning|
|---|---|
|`[abc]`|any one of a, b, or c|
|`[a-z]`|any lowercase letter|
|`[^a-z]`|anything except lowercase letters|
|`[0-9A-Z]`|digits + uppercase letters|
|`[\w]`|word characters|
|`[\s]`|whitespace|
|`[]]` or `\-`|escaping inside a class|
