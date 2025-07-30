---
tags: js, term, fundamental
---

An **array-like object** in JavaScript is an object that **resembles an array** but **does not inherit** from the `Array.prototype`. These objects have indexed elements and a `length` property but lack array methods like `.push()`, `.pop()`, etc.

---

## ✅ **Characteristics of Array-Like Objects**

1. Indexed elements (`0`, `1`, `2`, etc.)
2. A `length` property
3. **No** array methods (`map()`, `forEach()`, etc.)
4. Usually an `Object` type, not `Array`.

---

## 📊 **Examples of Array-Like Objects**

### 1. **Arguments Object**

Represents the arguments passed to a function.

```javascript
function showArgs() {
  console.log(arguments); // [Arguments] { '0': 'a', '1': 'b' }
  console.log(arguments.length); // 2
  console.log(arguments[0]); // 'a'
}
showArgs('a', 'b');
```

**✅ Convert to Array:**

```javascript
const argsArray = Array.from(arguments);
```

---

### 2. **NodeList / HTMLCollection**

Collections returned by DOM methods like `querySelectorAll()`.

```javascript
const divs = document.querySelectorAll('div');
console.log(divs); // NodeList(3) [div, div, div]
console.log(divs.length); // 3
console.log(divs[0]); // First div element
```

**✅ Convert to Array:**

```javascript
const divArray = Array.from(divs);
```

---

### 3. **Strings**

Strings are array-like because you can access characters by index.

```javascript
const str = 'hello';
console.log(str.length); // 5
console.log(str[0]); // 'h'
```

---

### 4. **Custom Array-Like Object**

You can manually create an object that mimics an array.

```javascript
const arrayLike = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3
};

console.log(arrayLike[0]); // 'a'
console.log(arrayLike.length); // 3
```

**✅ Convert to Array:**

```javascript
const realArray = Array.from(arrayLike);
console.log(realArray); // ['a', 'b', 'c']
```

---

## 🔎 **How to Detect Array-Like Objects**

1. Check for `length` and index properties:

```javascript
function isArrayLike(obj) {
  return obj != null && typeof obj === 'object' && 'length' in obj && obj.length >= 0;
}

console.log(isArrayLike([])); // true
console.log(isArrayLike({ 0: 'a', length: 1 })); // true
console.log(isArrayLike(123)); // false
```

2. Use `Array.isArray()` to distinguish from real arrays:

```javascript
console.log(Array.isArray([])); // true
console.log(Array.isArray(document.querySelectorAll('div'))); // false
```

---

## 📌 **When to Convert to an Array**

Use `Array.from()` or `Array.prototype.slice.call()` when you need to:

- Use array methods like `map()`, `filter()`, etc.
- Perform more complex operations on the collection.