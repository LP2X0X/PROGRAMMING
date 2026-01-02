---
tags: 
 - typescript
 - type
 - compound
---

A **tuple in TypeScript** is a special kind of array where:

- The **length is fixed**.
    
- The **types of elements at each index are known and ordered**.
    

It lets you model an array **with a precise structure**, similar to a small record.

---

## ✅ Basic Example

```ts
let person: [string, number] = ["Bob", 25];
```

Meaning:

- Index 0 must be a `string`
    
- Index 1 must be a `number`
    
- Length must be **exactly 2**
    

---

## ✅ Why use tuples?

### ✔ To represent structured, fixed-size data:

```ts
type HttpResponse = [number, string]; 
// [statusCode, message]
```

### ✔ To enforce order and type safety:

```ts
let point: [number, number] = [10, 20];
```

---

## ✅ Optional tuple elements

```ts
let user: [string, number?];
user = ["Alice"];
user = ["Alice", 30];
```

---

## ✅ Labels for readability (TS 4.0+)

```ts
let user: [name: string, age: number] = ["Bob", 25];
```

Labels don’t affect behavior—only readability.

---

## ✅ Rest elements in tuples (variadic tuples)

An open-ended tuple is where its items have some structure, but the number of elements isn't fixed.

```ts
type StringList = [string, ...string[]];

const a: StringList = ["first", "second", "third"];
```

---

## ✅ Readonly tuples

```ad-note
This is the prefered way of using tuple in TypeScript.
```

```ts
const point: readonly [number, number] = [1, 2];
// point[0] = 3; // ❌ Error
```

---

## ✅ Using tuples with functions

### **Return multiple values**

```ts
function useState(initial: number): [number, (n: number) => void] {
  let value = initial;
  const setValue = (n: number) => (value = n);
  return [value, setValue];
}
```

### React actually returns a tuple:

```ts
const [count, setCount] = useState(0);
```

---

## Summary

Tuples in TypeScript have:

- **Fixed order**
    
- **Fixed length (unless rest is used)**
    
- **Specific types for each position**
    
- Optional elements
    
- Readonly variants
    
- Labels for readability
    

They are perfect for situations where **you want a small, fixed-structure array** instead of a full object.