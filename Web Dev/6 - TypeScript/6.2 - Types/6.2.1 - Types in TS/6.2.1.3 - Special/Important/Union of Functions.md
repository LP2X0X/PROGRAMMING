---
tags: 
 - typescript
 - union
 - function
 - type
---

```ad-summary
When you have a **union of function types**, TypeScript only allows calling it with arguments that are valid for **every function in the union**, which often forces you to use narrowing or assertions even when the runtime logic is correct.
```

```ad-important
When functions are combined into a union, **their parameter types are intersected (must satisfy all signatures)**, while **their return types are unioned (may be any of the possible results)**.
```

---

# This is one case where union of functions can happen

```ad-note
If you want to **loop and call functions**, they must all share **the same callable signature**.
```

```ts
const logId = (obj: { id: string }) => {
  console.log(obj.id);
};
const logName = (obj: { name: string }) => {
  console.log(obj.name);
};

// Type of loggers
// const loggers: (((obj: {  
// id: string;  
// }) => void) | ((obj: {  
// name: string;  
// }) => void))[]
const loggers = [logId, logName];

// What must be the type of obj here
const logAll = (obj) => {
  loggers.forEach((func) => func(obj));
};
```

## 🔍 What “Loop and Call” Actually Means to TypeScript

When you write:

```ts
loggers.forEach((func) => func(obj));
```

TypeScript sees this as:

> “At runtime, `func` could be **any one** of the elements in `loggers`.”

There is **no narrowing** inside the loop.  
So `func` must be callable **without knowing which function it is**.

That forces this requirement:

> The argument must be valid for **every possible function** in the array.

## 🧩 Why a Union of Functions Breaks This Rule

Your array is inferred as:

```ts
(
  | ((obj: { id: string }) => void)
  | ((obj: { name: string }) => void)
)[]
```

This means:

- `func` is _either_ `logId`
    
- _or_ `logName`
    

TypeScript cannot choose one.

So it asks:

> “Is there **one argument type** that works for _both_ call signatures?”

## Yes, there is

```ts
const logId = (obj: { id: string }) => {
  console.log(obj.id);
};
const logName = (obj: { name: string }) => {
  console.log(obj.name);
};

const loggers = [logId, logName];
const logAll = (obj: { [key in "id" | "name"]: string }) => {
  loggers.forEach((func) => func(obj));
};
// Same as
const logAll = (obj: { id: string, name: string }) => {
  loggers.forEach((func) => func(obj));
};
```

This code works because you have manually satisfied the **Intersection Requirement** we discussed earlier.

By defining `obj` in `logAll` as a type that contains **both** `id` and `name`, you have created an argument that is safe for every function in the `loggers` array, regardless of which one is being called in the loop.

#### 1. The Intersection Logic

TypeScript looks at the loggers array and sees a union of function types:

$((obj: \{ id: string \}) \Rightarrow void) \mid ((obj: \{ name: string \}) \Rightarrow void)$

To call a function from this union safely, the argument must satisfy the intersection of the parameters:

$\{ id: string \} \cap \{ name: string \}$

Your definition of obj in logAll using the mapped type:

{ \[key in "id" | "name"]: string }

...is mathematically identical to:

{ id: string; name: string; }

Because this object has **both** properties, it is a valid "subtype" for both `logId` (which only requires `id`) and `logName` (which only requires `name`).

#### 2. Structural Compatibility

Remember that TypeScript is **structurally typed**.

- `logId` says: "I need an object with at least an `id` property."
    
- You provide: `{ id: "...", name: "..." }`.
    
- TypeScript says: "Great! It has an `id`. I don't care that it has an extra `name` property; I'll just ignore it."
    

This is why the code doesn't crash or error. You are providing **more** than what is required for each individual call, which is always safe in TypeScript.

#### 3. Visualizing the Call Stack inside `forEach`

| **Iteration** | **Function (func)** | **Required Property** | **Provided Property** | **Result**  |
| ------------- | ------------------- | --------------------- | --------------------- | ----------- |
| **1**         | `logId`             | `id`                  | `id` AND `name`       | ✅ **Safe** |
| **2**         | `logName`           | `name`                | `id` AND `name`       | ✅ **Safe** |

#### 4. What would break it?

If you changed `logAll` to use a **Union** instead of an intersection (Mapped Type), it would fail immediately:

```ts
// ❌ THIS WOULD FAIL
const logAll = (obj: { id: string } | { name: string }) => {
  loggers.forEach((func) => func(obj)); 
  // Error: Argument of type '{ id: string } | { name: string }' is not 
  // assignable to parameter of type '{ id: string } & { name: string }'.
};
```

In this failing case, `obj` might only have an `id`. If the loop happens to be on the `logName` function at that moment, the code would crash because `obj.name` would be `undefined`.

---

# Here's another one

```ts
const objOfFunctions = {
  string: (input: string) => input.toUpperCase(),
  number: (input: number) => input.toFixed(2),
  boolean: (input: boolean) => (input ? "true" : "false"),
};

const format = (input: string | number | boolean) => {
  const inputType = typeof input as "string" | "number" | "boolean";
  const formatter = objOfFunctions[inputType];

  return formatter(input as never);
};
```

## 🧠 Core Idea: You Are Indexing Into a **Union of Function Types**

### The object you defined

```ts
const objOfFunctions = {
  string: (input: string) => input.toUpperCase(),
  number: (input: number) => input.toFixed(2),
  boolean: (input: boolean) => (input ? "true" : "false"),
};
```

From TypeScript’s point of view, this is **not** a single polymorphic function.

Instead, it is **three different functions**, each with a distinct signature:

```ts
(input: string) => string
| (input: number) => string
| (input: boolean) => string
```

## 🔗 What Happens When You Index the Object

```ts
const formatter = objOfFunctions[inputType];
```

Because `inputType` is:

```ts
"string" | "number" | "boolean"
```

TypeScript computes:

```ts
objOfFunctions[inputType]
```

as:

```ts
(
  (input: string) => string
| (input: number) => string
| (input: boolean) => string
)
```

This is a **union of function types**, not a single callable signature.

## ⚠️ Why Union of Functions Is Hard to Call

TypeScript applies this rule:

> A value of type `A | B | C` can only be called if **every member** can be called with the same arguments.

But here:

| Function          | Accepts   |
| ----------------- | --------- |
| string formatter  | `string`  |
| number formatter  | `number`  |
| boolean formatter | `boolean` |

There is **no single argument type** that satisfies all three.

So TypeScript refuses this:

```ts
formatter(input); // ❌ unsafe
```

Because from the compiler’s perspective:

- `formatter` _might_ be the string formatter
    
- `input` _might_ be a number
    
- That would be a runtime error
    

## 🧩 Why the `as "string" | "number" | "boolean"` Cast Exists

```ts
const inputType = typeof input as "string" | "number" | "boolean";
```

This cast tells TypeScript:

> “Trust me — this `typeof` result matches the keys of `objOfFunctions`.”

Without it, `typeof input` is treated as the **full JS typeof union**, which would not index safely.

This cast is about **object indexing**, not function calling.

Here's the [[Normal typeof in TypeScript|reference]].

## 🧨 Why `input as never` Is Required

```ts
return formatter(input as never);
```

This looks strange, but it reveals the core issue.

### What `never` means here

`never` is the **intersection** of all parameter types:

```ts
string & number & boolean === never
```

By writing:

```ts
formatter(input as never)
```

You are telling TypeScript:

> “Assume the argument satisfies _all_ function signatures.”

This silences the error because `never` is assignable to every type. It acts like a [[Type Assertion| type assertion]]  in this case.

⚠️ Important:

- This does **not** make the code safer
    
- It only bypasses the compiler’s check
    
- You are asserting something TS cannot prove
    

## 🧠 Key Concept to Internalize

### This code works at runtime because:

- `typeof input` and `input` are correlated
    
- The correct formatter is selected
    
- JavaScript guarantees consistency
    

### But TypeScript cannot encode that correlation automatically

So from TS’s perspective:

- You indexed into a union
    
- You got a union of functions
    
- You tried to call it
    
- There is no shared callable signature
    

## 📌 Why This Is NOT a Discriminated Union

Discriminated unions work when:

- The **value and discriminator live in the same object**
    
- TypeScript can track the relationship
    

Here, the discriminator (`typeof input`) is **computed**, not structural.

That breaks the narrowing chain.

## ✅ The Rule to Remember

> **When you index an object with a union key, you get a union of values.  
> When those values are functions, they are only callable if their parameter types align.**

Your example is a textbook illustration of that rule.

## 🧾 Final Summary

- `objOfFunctions[inputType]` produces a **union of function types**
    
- Unioned functions are only callable with **common parameters**
    
- Your functions share no common input type
    
- `as never` forces compatibility by assertion
    
- The runtime logic is correct; the type system cannot prove it
    

This is not a bug or limitation—it is TypeScript being **sound and conservative by design**.