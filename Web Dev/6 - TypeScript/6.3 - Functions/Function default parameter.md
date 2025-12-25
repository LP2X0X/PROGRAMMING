---
tags: 
 - typescript
 - function
 - parameter
---

### 🔹 What is a default parameter?

A **default parameter** provides a fallback value when:

- the argument is **omitted**
    
- or explicitly passed as `undefined`
    

```ts
function greet(name = "Guest") {
  return `Hello, ${name}`;
}
```

---

### 🔹 When the default is used ✅

```ts
greet();              // "Hello, Guest"
greet(undefined);     // "Hello, Guest"
```

❌ **Not used when `null` is passed**

```ts
greet(null); // Type error (unless `name: string | null`)
```

---

### 🔹 Type inference 🧩

TypeScript **infers the parameter type** from the default value:

```ts
function repeat(count = 3) {
  // count is number
}
```

Equivalent to:

```ts
function repeat(count: number = 3) {}
```

---

### 🔹 Default vs optional (`?`) ⚖️

```ts
function a(x?: number) {}
function b(x = 10) {}
```

|Feature|`x?: number`|`x = 10`|
|---|---|---|
|Can be omitted|✅|✅|
|Can be `undefined`|✅|✅|
|Has runtime fallback|❌|✅|
|Type includes `undefined`|✅|❌|

👉 **Default parameters are safer** when you need a guaranteed value.

---

### 🔹 Order rule 📏

Default parameters must come **after required ones**:

```ts
function ok(a: string, b = "x") {}
```

❌ Invalid:

```ts
function bad(a = "x", b: string) {}
```

---

### 🔹 With destructuring 📦

```ts
function createUser({ role = "user" }: { role?: string }) {
  return role;
}
```

---

### 🔹 Runtime behavior ⚙️

Default parameters:

- Exist in **JavaScript output**
    
- Are applied **at call time**
    
- Are **not TypeScript-only**
    

---

### ⚠️ Common pitfall

```ts
function log(msg = "") {}
log(undefined); // uses default
log(null);      // error (unless allowed)
```

---

### 🧠 Mental model

> 🟢 **Optional (`?`)** = “might be missing”  
> 🟢 **Default (`=`)** = “will always have a value”

---

### ✅ Best practice

Use **default parameters** when:

- you want predictable values
    
- you want to avoid `undefined` checks
    
- you care about runtime safety
    