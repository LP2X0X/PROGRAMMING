---
tags: 
 - regex
 - term
---

Metacharacters are **special symbols** in a regular expression that **do not represent themselves literally**.  
Instead, they control how the regex engine matches text.

They change **pattern meaning**, such as matching any character, repeating patterns, creating groups, or defining boundaries.

---

# **📌 Most Common Regex Metacharacters (with short meaning)**

### **Character Matching**

|Symbol|Meaning|
|---|---|
|`.`|Match **any single character** except newline|
|`\d`|Match **digit** (0–9)|
|`\D`|Match **non-digit**|
|`\w`|Match **word character** (`a–z`, `A–Z`, `0–9`, `_`)|
|`\W`|Match **non-word character**|
|`\s`|Match **whitespace** (space, tab, newline)|
|`\S`|Match **non-whitespace**|

---

### **Quantifiers (repetition)**

|Symbol|Meaning|
|---|---|
|`*`|**0 or more** repetitions|
|`+`|**1 or more** repetitions|
|`?`|**0 or 1** (optional)|
|`{n}`|Exactly **n** times|
|`{n,}`|**n or more** times|
|`{n,m}`|Between **n and m** times|

---

### **Anchors (position, not character)**

|Symbol|Meaning|
|---|---|
|`^`|Start of string|
|`$`|End of string|
|`\b`|Word boundary|
|`\B`|Not a word boundary|

---

### **Grouping & Alternation**

|Symbol|Meaning|
|---|---|
|`()`|Capturing group|
|`(?: )`|Non-capturing group|
|`|`|

---

### **Character Sets**

|Symbol|Meaning|
|---|---|
|`[ ]`|A set of allowed characters|
|`[^ ]`|A **negated** character set|
|`[a-z]`|Character **range**|

---

### **Escaping**

|Symbol|Meaning|
|---|---|
|`\`|Escape next character so it's treated literally (e.g., `\.` matches a dot)|

---

# **📌 Quick Regular Use Cases**

### **Digits**

```
/\d+/      → one or more digits
```

### **Email**

```
/\w+@\w+\.\w+/
```

### **Trim extra spaces**

```
/\s+/
```

### **Match a word**

```
/\bword\b/
```

### **Optional pattern**

```
/hello!?/
```
