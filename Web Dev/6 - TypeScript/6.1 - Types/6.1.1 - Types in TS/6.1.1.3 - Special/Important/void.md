---
tags: 
 - typescript
 - type
---

## 🧩 **1. What is `void`?**

- `void` represents **the absence of a value**.
    
- Usually seen in **function return types**.
    
- If a function doesn’t return anything → TypeScript infers `void`.
    

```ts
function logMessage(): void {
  console.log("Hello");
}
```

---

## 🧩 **2. What value can `void` hold?**

- In **TypeScript**: only `undefined`
    
- In **JavaScript**: technically `undefined` is the default value of a void function
    

```ts
let a: void = undefined; // valid
let b: void = null;      // ❌ not allowed unless strictNullChecks is off
```

---

## 🧩 **3. What `void` _is not_**

- It is **not** the same as:
    
    - `undefined` (a value)
        
    - `null` (a value)
        
    - `never` (unreachable code / no return ever)
        
- It does **not** mean “no type”—it means “there is a type, but it returns nothing”.
    

---

## 🧩 **4. Primary use case: function return type**

### ✔ Functions used only for side effects

(no return → inferred `void`)

```ts
function greet(): void {
  console.log("Hello!");
}
```

### ✔ Explicitly marking: “This function shouldn’t return anything”

Useful in public APIs:

```ts
function onClick(callback: () => void) {}
```

---

## 🧩 **5. `void` vs `never`**

### `void`

> The function can finish normally but doesn’t return anything.

```ts
function print(): void {
  console.log("x");
}
```

### `never`

> The function can **never** finish normally.

- throws errors
    
- infinite loops
    

```ts
function fail(): never {
  throw new Error("Fail!");
}
```

---

## 🧩 **6. `void` in callbacks**

A callback typed as `() => void` is allowed to return a value —  
but the caller **must ignore it**.

Example:

```ts
function run(cb: () => void) {
  cb(); // return value ignored
}

run(() => 123); // valid
```

Why?  
To support patterns like `.forEach()`:

```ts
[1, 2, 3].forEach(n => n * 2); // returning something is allowed
```

---

## 🧩 **7. `void` in Promises**

### ⚠️ Important difference:

`Promise<void>`  
→ Promise that resolves with **no meaningful value**

```ts
async function save(): Promise<void> {
  await apiCall();
}
```

You still _must_ use `Promise<void>` for async functions that don't return data.

---

## 🧩 **8. Useful real-world scenarios**

### ✔ Event handlers

```ts
const handleClick = (e: MouseEvent): void => {
  console.log("clicked");
};
```

### ✔ Utility APIs that don’t return

```ts
function logError(message: string): void {
  console.error(message);
}
```

### ✔ Cleanup functions in React

```ts
useEffect(() => {
  return (): void => {
    console.log("cleanup");
  };
}, []);
```

### ✔ Command-like functions

(e.g., saving, deleting, writing to a DB)

---

## 🧩 **9. When NOT to use void**

Do not use `void`:

- For variables (almost never helpful)
    
- When you need to return something later
    
- As a union type (e.g., `string | void` is discouraged)
    

---

# 📌 **Summary (Quick Notes)**

|Concept|Meaning|
|---|---|
|`void`|function returns no value|
|Allowed values|only `undefined`|
|Common use|callbacks, event handlers, side-effect functions|
|`void` vs `never`|`void` ends normally; `never` never ends|
|With async|use `Promise<void>`|
|Callback quirk|callbacks can return something, but caller ignores it|
