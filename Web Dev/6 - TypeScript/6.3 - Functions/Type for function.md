---
tags: 
 - typescript
 - type
---

## 🧠 **Function Types in TypeScript**

Function types describe **what arguments a function accepts** and **what it returns**. In TypeScript, functions are first-class values, so they have _explicit, expressive types_ just like objects and primitives.

---

## ✍️ **Ways to Type a Function**

### 🟢 Function Declaration

```ts
function add(a: number, b: number): number {
  return a + b;
}
```

**Key points**

- Parameter types are required for safety
    
- Return type is inferred, but can be explicit
    
- Best for named, reusable logic
    

---

### 🟢 Function Expression

```ts
const add = (a: number, b: number): number => a + b;
```

Equivalent to a declaration, but:

- Return type is inferred from the expression
    
- Common in functional and React code
    

---

### 🟢 Function Type Alias

```ts
type AddFn = (a: number, b: number) => number;

const add: AddFn = (a, b) => a + b;
```

Use when:

- The same function shape is reused
    
- Passing functions as arguments
    
- Defining APIs
    

---

## 📥 **Parameter Typing Rules**

### 🔹 Required vs Optional Parameters

```ts
function log(message: string, level?: "info" | "error") {}
```

- Optional parameters must come **after required ones**
    
- Optional parameters are `T | undefined`
    

---

### 🔹 Default Parameters

```ts
function greet(name: string = "Guest") {}
```

- Defaults implicitly make parameters optional
    
- Type is inferred from the default value
    

---

### 🔹 Rest Parameters

```ts
function sum(...nums: number[]): number {
  return nums.reduce((a, b) => a + b, 0);
}
```

- Must be an array type
    
- Only one rest parameter allowed
    

---

## 🔄 **Return Type Behavior**

### 🟢 Inference

```ts
function isEven(n: number) {
  return n % 2 === 0;
}
// boolean
```

TypeScript infers return types automatically.

---

### ⚠️ When to Explicitly Annotate

```ts
function parse(input: string): number | null {
  if (input === "") return null;
  return Number(input);
}
```

Explicit return types help:

- Public APIs
    
- Complex logic
    
- Prevent accidental changes
    

---

### ❌ `void` vs `never`

```ts
function log(msg: string): void {
  console.log(msg);
}

function fail(msg: string): never {
  throw new Error(msg);
}
```

- `void`: function finishes normally
    
- `never`: function never returns
    

---

## 🧩 **Function Type Compatibility**

TypeScript uses **structural typing**.

```ts
type Handler = (value: string) => void;

const fn = (v: string) => {};
```

- Parameter names do not matter
    
- Parameter count must be compatible
    
- Return type must match or be narrower
    

---

### 🔄 Parameter Bivariance (Special Case)

```ts
type Listener = (event: MouseEvent) => void;
```

Used in callbacks (e.g. DOM, React) for ergonomics, even though it is technically unsound.

---

## 🧠 **Overloads**

Use overloads when a function has **multiple call signatures**.

```ts
function format(value: string): string;
function format(value: number): string;
function format(value: string | number) {
  return String(value);
}
```

Rules:

- Overload signatures are public
    
- Implementation signature is private
    
- Narrowing happens in the implementation
    

---

## 🧪 **Generics in Functions**

```ts
function identity<T>(value: T): T {
  return value;
}
```

Generics:

- Preserve type information
    
- Avoid `any`
    
- Enable inference from arguments
    

---

### 🔒 Generic Constraints

```ts
function getLength<T extends { length: number }>(v: T) {
  return v.length;
}
```

---

## 🧩 **this Parameter in Functions**

```ts
function greet(this: { name: string }) {
  console.log(this.name);
}
```

- `this` is a _fake parameter_
    
- Used only for type checking
    
- Common in class-free APIs
    

---

## 🔐 **Async Function Types**

```ts
async function fetchData(): Promise<string> {
  return "data";
}
```

- Return type is always `Promise<T>`
    
- Use `Awaited<T>` for unwrapping
    

---

## ⚠️ **Common Pitfalls**

### ❌ Overusing `any`

```ts
const fn: any = () => {};
```

Avoid — it disables type checking.

---

### ❌ Mismatched Return Types

```ts
function test(): number {
  return "x"; // error
}
```

---

## 🧠 **Mental Model**

> _A function type is a contract:_  
> **inputs → behavior → output**

TypeScript ensures:

- Callers respect the contract
    
- Implementations fulfill the promise
    