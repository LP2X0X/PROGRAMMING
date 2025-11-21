---
tags: 
 - js
 - object
 - static
 - method
---

## 🔍 **What is `Object.fromEntries()`?**

`Object.fromEntries()` converts a list of key–value pairs (entries) **into an object**.

Think of it as the opposite of:

- `Object.entries(obj)` → converts object → array of entries
    
- `Object.fromEntries(entries)` → converts entries → object
    

---

## 📌 **Basic usage**

```js
const entries = [
  ["name", "Long"],
  ["age", 25]
];

const obj = Object.fromEntries(entries);

console.log(obj);
// { name: "Long", age: 25 }
```

---

## 💡 **Where it becomes useful (real dev cases)**

### 1. **Turn a `Map` into an object**

```js
const map = new Map();
map.set("a", 1);
map.set("b", 2);

const obj = Object.fromEntries(map);
```

---

### 2. **Convert form data (React Router / controlled forms)**

This is one of the **most practical uses** in React:

```js
const formData = new FormData(e.target);
const data = Object.fromEntries(formData);

console.log(data);
// { username: "long", password: "1234" }
```

Perfect for:

- handling submit events
    
- sending clean JSON to APIs
    
- debugging form structures
    

---

### 3. **Filtering an object**

Instead of writing messy loops:

```js
const user = { name: "A", age: 20, active: true };

// keep only name + active
const filtered = Object.fromEntries(
  Object.entries(user).filter(([key]) => key !== "age")
);

console.log(filtered);
// { name: "A", active: true }
```

---

### 4. **Transforming keys or values**

```js
const user = { name: "A", points: 10 };

const updated = Object.fromEntries(
  Object.entries(user).map(([key, val]) => [key, val * 2])
);

console.log(updated);
// { name: "A", points: 20 }
```

---

### 5. **Remove undefined values (clean API JSON)**

```js
const obj = {
  name: "A",
  email: undefined,
  age: 25
};

const cleaned = Object.fromEntries(
  Object.entries(obj).filter(([_, val]) => val !== undefined)
);

// { name: "A", age: 25 }
```

---

## 🔄 **Paired with `Object.entries()`**

#### Convert object → entries → modify → object

```js
const user = { name: "A", age: 20 };

const result =
  Object.fromEntries(
    Object.entries(user).map(([key, value]) => [key.toUpperCase(), value])
  );

console.log(result);
// { NAME: "A", AGE: 20 }
```

---

## ⚡ Cheatsheet

|Operation|Method|
|---|---|
|object → entries|`Object.entries(obj)`|
|entries → object|`Object.fromEntries(entries)`|
|transform values|use `.map()` inside|
|filter properties|use `.filter()` inside|
|data from form|`Object.fromEntries(new FormData(form))`|
|map → object|`Object.fromEntries(map)`|