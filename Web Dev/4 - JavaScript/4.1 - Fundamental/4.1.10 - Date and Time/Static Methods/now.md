---
tags: 
 - js
 - Date
 - static
 - methods
---

`Date.now()` is a **static method on the `Date` class**, meaning:

- You **call it on `Date`**, not on a date object.
    
- It **returns the current timestamp** in milliseconds.
    
- It’s equivalent to:
    

```js
new Date().getTime()
```

### Example

```js
const ts = Date.now(); // ✓ works
```

### ❌ Invalid

```js
const d = new Date();
d.now(); // ✗ TypeError: d.now is not a function
```

---

# All Static Methods of `Date` (Complete List)

JavaScript’s `Date` actually has _very few_ static methods:

### ✔️ `Date.now()`

Returns timestamp (ms since Unix epoch).

### ✔️ `Date.parse(string)`

Parses a date string → returns timestamp.

```js
Date.parse("2025-01-01");
```

### ✔️ `Date.UTC(year, month, ...)`

Creates a timestamp using UTC values.

```js
Date.UTC(2025, 0, 1); // Jan = 0
```

---

# Instance Methods (NOT static)

These require:

```js
const d = new Date();
```

Examples:

```js
d.getHours();
d.getMinutes();
d.toISOString();
d.setFullYear(2025);
```

---

# Summary

|Method|Static?|Usage|
|---|---|---|
|`Date.now()`|✅ Yes|`Date.now()`|
|`Date.parse()`|✅ Yes|`Date.parse("...")`|
|`Date.UTC()`|✅ Yes|`Date.UTC(...)`|
|Everything else (`getTime`, `getHours`, etc.)|❌ No|`new Date().getHours()`|