### 📚 **Object Coercion in JavaScript**

**Object coercion** refers to the process where JavaScript converts an object into a primitive value (`string`, `number`, or `boolean`) when an operation requires it.

---

### ✅ **When Does Object Coercion Happen?**

Object coercion occurs when:

1. You perform operations involving **strings** (e.g., concatenation).
2. Use **numeric contexts** (e.g., arithmetic operations).
3. Compare objects with **loose equality (`==`)**.

---

### 🔍 **How Object Coercion Works**

JavaScript uses the following methods in order:

1. **`Symbol.toPrimitive()`** – If defined, it has the highest priority.
2. **`valueOf()`** – Tries to return a primitive value.
3. **`toString()`** – Converts the object to a string if `valueOf()` fails.

---

### 📊 **Example 1: Default Coercion Behavior**

```javascript
const obj = {
  valueOf() {
    return 42;
  },
  toString() {
    return 'Hello';
  },
};

console.log(obj + 1); // 43 (valueOf() is used)
console.log(String(obj)); // "Hello" (toString() is used)
```

👉 **Numeric operations** trigger `valueOf()` first.  
👉 **String contexts** (e.g., `String(obj)`) prefer `toString()`.

---

### 📌 **Explicit Coercion Methods**

1. **`String(obj)`** – Converts to a string.
2. **`Number(obj)`** – Converts to a number.
3. **`Boolean(obj)`** – Converts to a boolean.

```javascript
const obj = { a: 1 };

console.log(String(obj)); // "[object Object]"
console.log(Number(obj)); // NaN (no numeric value found)
console.log(Boolean(obj)); // true (all objects are truthy)
```

---

### 🛠️ **Customizing Object Coercion**

You can define custom coercion using `Symbol.toPrimitive()`.

```javascript
const obj = {
  [Symbol.toPrimitive](hint) {
    if (hint === 'number') return 42;
    if (hint === 'string') return 'Custom String';
    return null;
  },
};

console.log(obj + 1); // 43 (number hint)
console.log(String(obj)); // "Custom String" (string hint)
console.log(obj == null); // true (default hint)
```

- `"string"`: For string contexts.
- `"number"`: For numeric contexts.
- `"default"`: For other cases (e.g., loose equality).

---

### 🔢 **Example 2: Coercion in Comparisons**

```javascript
const obj = {
  valueOf() { return 10; }
};

console.log(obj > 5); // true (valueOf() is called)
console.log(obj == 10); // true (valueOf() is called)
```

---

### 🚨 **Common Pitfalls**

1. **Implicit coercion** can lead to **unexpected behavior**:

```javascript
console.log([] + {}); // "[object Object]"
console.log({} + []); // 0 (due to automatic semicolon insertion)
```

2. Objects in boolean contexts always **coerce to `true`**:

```javascript
if ({}) console.log('Always truthy'); // "Always truthy"
```

Would you like a deeper dive into edge cases or advanced coercion behavior? 😊