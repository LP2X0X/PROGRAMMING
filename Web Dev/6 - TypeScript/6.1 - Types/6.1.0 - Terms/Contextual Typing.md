---
tags: 
 - typescript
 - type
---

## 🧩 1. What is contextual typing?

> **Contextual typing** means TypeScript _infers_ the type of an expression **from its context** — such as the variable it’s assigned to, the function parameter it’s passed into, or the event handler it’s used as.

In short:

> The **surrounding code provides the expected type**, and TypeScript fills it in automatically.

---

### 🔹 Example 1: Function assigned to a typed variable

```ts
type Greeter = (name: string) => void;

const greet: Greeter = (n) => {
  console.log("Hello " + n.toUpperCase()); // ✅ n is inferred as string
};
```

➡️ Here, `(n)` has no type annotation —  
but because `greet`’s context expects a `(string) => void` function, TypeScript infers `n: string`.

---

### 🔹 Example 2: Event handlers in DOM

```ts
document.addEventListener("click", (event) => {
  console.log(event.clientX); // ✅ inferred as MouseEvent
});
```

➡️ You never typed `event`,  
but TypeScript knows that for `"click"`, the handler receives a `MouseEvent`.

---

### 🔹 Example 3: Array literals

```ts
let numbers = [1, 2, 3];  // inferred as number[]
let mixed = [1, "a"];     // inferred as (string | number)[]
```

➡️ The **array literal’s context** (the element types) drives inference.

---

## 🧠 2. Why contextual typing matters

Without contextual typing, you’d need to annotate everything manually:

```ts
const greet = (n: string): void => console.log(n.toUpperCase());
```

With it, TypeScript keeps your code **cleaner and more ergonomic** — while staying safe.

---

## ⚙️ 3. Common contextual typing contexts

|Context|Example|What TS infers|
|---|---|---|
|Variable assignment|`const f: (x: number) => void = x => {}`|`x: number`|
|Function argument|`array.map(x => x * 2)`|`x` inferred from array element|
|Return position|`function make(): number { return 5; }`|Return type|
|Object literal|`{ onClick: e => console.log(e.clientX) }`|`e` inferred as MouseEvent|
|JSX event handler|`<button onClick={e => e.preventDefault()} />`|`e` inferred as React.MouseEvent|

---

## 🪤 4. Common pitfalls

### ⚠️ a. Missing context

If TypeScript can’t find context, it falls back to `any`.

```ts
const fn = (x) => x.toUpperCase(); // x: any
```

✅ Fix by giving context:

```ts
const fn: (x: string) => string = (x) => x.toUpperCase();
```

---

### ⚠️ b. Context loss through intermediate variables

Context doesn’t “flow” across assignments unless typed.

```ts
const handler = (e) => console.log(e.clientX); // e: any ❌
document.addEventListener("click", handler);   // no context left
```

✅ Fix:

```ts
const handler: (e: MouseEvent) => void = (e) => console.log(e.clientX);
```

or inline it:

```ts
document.addEventListener("click", (e) => console.log(e.clientX)); // ✅ inferred
```

---

### ⚠️ c. Context limited to one side of assignment

Type inference doesn’t go _both_ ways:

```ts
let a = (x: number) => 0;
let b = (x) => "hi";
b = a; // ❌ type mismatch — context doesn’t merge
```

---

## 💡 5. Contextual typing + union types

When context is ambiguous, TypeScript infers the **most general** type:

```ts
let f = Math.random() > 0.5
  ? (x: string) => x.toUpperCase()
  : (x: number) => x.toFixed();
f("hi"); // ❌ Error: 'string | number' not assignable to 'string'
```

TypeScript infers `f: (x: string | number) => string | number`

✅ Use explicit overloads or type guards if you need more precision.

---

## 🚀 6. Contextual typing in callbacks

Super common in array methods and React components:

```ts
const nums = [1, 2, 3];
nums.forEach((n) => console.log(n.toFixed())); // n inferred as number
```

```tsx
type Props = { onChange: (value: string) => void };
const Input = (props: Props) => <input onChange={e => props.onChange(e.target.value)} />;
```

➡️ `e` is inferred as `React.ChangeEvent<HTMLInputElement>` thanks to JSX contextual typing.

---

## 🧭 7. TL;DR Cheatsheet

|Concept|Description|Example|
|---|---|---|
|Contextual typing|Type inferred from usage context|`const f: (n:number)=>void = n=>{}`|
|Works with|Function params, callbacks, event handlers, object literals|`array.map(x=>x*2)`|
|Context lost when detached|Assign to variable → no inference|`const f=(e)=>...` then use later|
|Fix|Add explicit type or inline expression|`const f=(e:MouseEvent)=>...`|
|Benefit|Cleaner, safer code|Avoid repetitive type annotations|

---

### 🧠 Summary

> **Contextual typing = direction of inference.**  
> TypeScript doesn’t just look _inside_ an expression — it also looks _around_ it to decide types.

It’s one of the main reasons TypeScript code feels both **strongly typed and lightweight**.