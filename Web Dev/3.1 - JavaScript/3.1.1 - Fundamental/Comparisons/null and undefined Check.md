---
tags: js, comparison
---

In JavaScript, `null` and `undefined` are two distinct types that both represent "no value", but they behave differently. Here's a **clear comparison and checklist**:

---

### 🔹 1. **Type**

```js
typeof null         // "object"  ❗ (this is a long-standing JS quirk)
typeof undefined    // "undefined"
```

---

### 🔹 2. **Equality Comparison**

#### 🟡 Loose equality (`==`)

```js
null == undefined   // ✅ true (they're treated as equal)
```

#### 🔴 Strict equality (`===`)

```js
null === undefined  // ❌ false (different types)
```

---

### 🔹 3. **Default/Unset Use Cases**

|When it appears|`null`|`undefined`|
|---|---|---|
|Manually set value|✅ Yes|🟡 Rarely (usually avoided)|
|Uninitialized variable|❌ No|✅ Yes|
|Missing function arg|❌ No|✅ Yes|
|Missing object prop|❌ No|✅ Yes|
|JSON|✅ Included as `null`|❌ Omitted if undefined|

---

### 🔹 4. **Falsy Values**

Both are falsy:

```js
Boolean(null)        // false
Boolean(undefined)   // false
```

---

### 🔹 5. **Practical Checking Tips**

```js
if (value == null) {
  // ✅ true if value is either null or undefined
}

if (value === null) {
  // ✅ true only if null
}

if (value === undefined) {
  // ✅ true only if undefined
}
```

✅ **Use `== null`** when you want to check for **both null and undefined** at once.  
❗ Avoid assigning `undefined` manually — use `null` for intentional "no value".
