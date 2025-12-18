---
tags: 
 - js
 - object
 - static
 - method
---

## 1. Definition

**Object.entries()** is a static method that returns an array of a given object's **own enumerable string-keyed properties**, in the form of `[key, value]` pairs.

```
Object.entries(obj)
```

---

## 2. Return Value

Returns a **new array** of arrays:

```
[
  [key1, value1],
  [key2, value2],
  ...
]
```

Keys are always **strings** (or symbols converted to strings), and the order follows the standard property enumeration order of objects.

---

## 3. What It Includes

- Only **own** properties  
    (not inherited through prototype)
    
- Only **enumerable** properties  
    (non-enumerable properties are skipped)
    
- Only **string-keyed** properties  
    (symbol-keyed properties are ignored)
    

---

## 4. Typical Use Cases

### a. Loop through object properties

Object.entries is often used with destructuring inside a loop:

```js
const user = { name: "Long", age: 20 };

for (const [key, value] of Object.entries(user)) {
  console.log(key, value);
}
```

### b. Convert objects to Maps

Maps accept entries directly:

```js
const obj = { a: 1, b: 2 };
const map = new Map(Object.entries(obj));
```

### c. Convert entries back to an object

(Useful when filtering or transforming)

```js
const filtered = Object.fromEntries(
  Object.entries(obj).filter(([k, v]) => v > 1)
);
```

### d. Working with JSON-like objects

Entry-based transformations are common when manipulating structured data.

---

## 5. Comparison with Related Methods

| Method                 | Returns                            | Notes                  |
| ---------------------- | ---------------------------------- | ---------------------- |
| `Object.keys(obj)`     | `["key1", "key2"]`                 | Only keys              |
| `Object.values(obj)`   | `[value1, value2]`                 | Only values            |
| `Object.entries(obj)`  | `[[key1, value1], [key2, value2]]` | Keys + values          |
| `Object.fromEntries()` | Converts `[[key, value]]` → object | Reverse of `entries()` |

---

## 6. Ordering Rules

JavaScript defines property enumeration order as:

1. **Integer-like keys** in ascending order
    
2. **String keys** in insertion order
    
3. **Symbol keys** (ignored by Object.entries)
    

Example:

```js
Object.entries({ 2: "b", 1: "a", x: 3 })
// → [["1","a"], ["2","b"], ["x",3]]
```

---

## 7. Edge Cases

### a. Works with arrays

Indexes become keys:

```js
Object.entries(["a", "b"]);
// → [["0", "a"], ["1", "b"]]
```

### b. Ignores inherited properties

```js
const parent = { x: 1 };
const child = Object.create(parent);
child.y = 2;

Object.entries(child); // → [["y", 2]]
```

### c. Ignores non-enumerable properties

```js
const obj = {};
Object.defineProperty(obj, "hidden", {
  value: 123,
  enumerable: false
});

Object.entries(obj); // → []
```

---

## 8. Time Complexity

The method is essentially **O(n)** with respect to the number of enumerable properties.
