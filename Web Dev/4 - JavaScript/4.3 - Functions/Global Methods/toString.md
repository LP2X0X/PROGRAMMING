---
tags: 
 - function
 - global
---

👉 **`toString()` does behave differently depending on the object type** — it’s a _method that each type can override_ to produce a meaningful string form of itself.

---

## 🔹 1. `toString()` is not one single function

In JavaScript, every object inherits a `toString()` method, but each **type (class or constructor)** can **override** it to define its own behavior.

So:

```js
something.toString()
```

depends on **what `something` is**.

---

## 🔹 2. Examples by type

### 🧮 Numbers / BigInts

They support a numeric _base conversion_ form:

```js
(255).toString(16);  // "ff"
(17n).toString(2);   // "10001"
```

- The optional argument is **radix** (2–36).
    
- Converts the numeric value to that base.
    

---

### 📜 Strings

For strings, `.toString()` just returns itself:

```js
"hello".toString(); // "hello"
```

Because it’s already a string.

---

### 🧱 Arrays

For arrays, `.toString()` joins elements with commas:

```js
[1, 2, 3].toString(); // "1,2,3"
```

Equivalent to `.join(",")`.

---

### ⚙️ Objects

For plain objects, the **default** is very generic:

```js
({a: 1}).toString(); // "[object Object]"
```

That’s inherited from `Object.prototype.toString`.

However, custom classes can override it:

```js
class Point {
  constructor(x, y) { this.x = x; this.y = y; }
  toString() { return `(${this.x}, ${this.y})`; }
}

new Point(2, 3).toString(); // "(2, 3)"
```

---

### 🧩 Dates

`Date.toString()` gives a readable time:

```js
new Date().toString();
// "Thu Oct 09 2025 13:48:20 GMT+0700 (Indochina Time)"
```

---

## 🔹 3. Summary Table

|Type|Example|`toString()` Result|
|---|---|---|
|**Number**|`(15).toString(2)`|`"1111"`|
|**BigInt**|`(15n).toString(2)`|`"1111"`|
|**String**|`"abc".toString()`|`"abc"`|
|**Array**|`[1,2,3].toString()`|`"1,2,3"`|
|**Object**|`{a:1}.toString()`|`"[object Object]"`|
|**Date**|`new Date().toString()`|`"Thu Oct 09 2025 ..."`|
|**Custom Class**|custom override|whatever you define|

---

✅ **In short:**  
`toString()` is a _universal_ method, but each type defines **its own meaning** for how it turns into text.