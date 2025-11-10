---
tags: 
 - typescript
 - narrowing
---

## 🧠 What Is Narrowing?

**Narrowing** is the process TypeScript uses to **refine a variable’s type** from a broad type (like `string | number`) to a **more specific** one (like just `string`) based on logic in your code.

> 💬 Think of it like TypeScript saying:  
> “After this check, I can be more confident about what this value actually is.”

---

### 🧩 Example

```ts
function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase()); // id: string
  } else {
    console.log(id); // id: number
  }
}
```

Here:

- Before the `if`, `id` is `string | number`
    
- After the `typeof` check, TS _narrows_ it to `string` inside the `if`
    
- And to `number` in the `else`
    

---

## ⚙️ How Narrowing Works

TypeScript uses **type guards** (checks in your code) to infer narrower types automatically.

---

## 🧱 Common Narrowing Techniques

### 1️⃣ **typeof Narrowing**

Used for primitive types.

```ts
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + value;
  }
  return padding + value;
}
```

✅ Works with:  
`"string"`, `"number"`, `"boolean"`, `"symbol"`, `"bigint"`, `"undefined"`, `"object"`, `"function"`

---

### 2️⃣ **Truthiness Narrowing**

Based on JavaScript truthy / falsy checks.

```ts
function printMessage(msg?: string) {
  if (msg) {
    console.log(msg.toUpperCase()); // msg: string
  } else {
    console.log("No message");
  }
}
```

⚠️ Be aware that empty strings, `0`, and `false` are falsy — sometimes that matters.

---

### 3️⃣ **Equality Narrowing**

TypeScript can narrow types when you compare them with `===`, `!==`, etc.

```ts
function compare(a: string | number, b: string | boolean) {
  if (a === b) {
    // a and b both have to be string here
    console.log(a.toUpperCase());
  }
}
```

---

### 4️⃣ **`in` Operator Narrowing**

Useful for checking property existence in **object unions**.

```ts
type Bird = { fly: () => void };
type Fish = { swim: () => void };

function move(animal: Bird | Fish) {
  if ("fly" in animal) {
    animal.fly(); // Bird
  } else {
    animal.swim(); // Fish
  }
}
```

---

### 5️⃣ **`instanceof` Narrowing**

Used with class instances.

```ts
class DateRange {
  start: Date;
  end: Date;
}

function logDate(date: Date | DateRange) {
  if (date instanceof Date) {
    console.log(date.toUTCString()); // Date
  } else {
    console.log(`${date.start} - ${date.end}`); // DateRange
  }
}
```

---

### 6️⃣ **Type Predicates**

Used to create **custom type guards** — functions that tell TS how to narrow types.

```ts
type Cat = { meow: () => void };
type Dog = { bark: () => void };

function isCat(animal: Cat | Dog): animal is Cat {
  return (animal as Cat).meow !== undefined;
}

function speak(animal: Cat | Dog) {
  if (isCat(animal)) {
    animal.meow(); // Cat
  } else {
    animal.bark(); // Dog
  }
}
```

💡 `animal is Cat` is the **type predicate** — it tells TypeScript what type to assume if the function returns `true`.

---

### 7️⃣ **Discriminated Union Narrowing**

This is the _most powerful_ narrowing pattern — great for managing multiple variants of objects.

```ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "triangle"; base: number; height: number };

function area(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    case "triangle":
      return (shape.base * shape.height) / 2;
  }
}
```

✅ TS uses the **`kind`** field as a **discriminant** to narrow automatically inside each `case`.

---

## 🧩 Control Flow Narrowing

TypeScript also tracks variable changes through the control flow of your code — it’s **flow-sensitive typing**.

```ts
let val: string | null = "hello";
val = null;

if (val) {
  val.toUpperCase(); // unreachable, so val never narrowed
}
```

TS keeps track of reassignment and scopes to maintain accurate narrowing.

---

## 🪄 Tips & Good Practices

|Tip|Description|
|---|---|
|✅ Prefer `as` little as possible|Let TypeScript infer with narrowing instead of forcing with assertions|
|🧭 Use discriminated unions|For complex branching logic (like Redux actions, shape variants)|
|🔒 Always enable `strictNullChecks`|Makes truthiness narrowing more predictable|
|💡 Use type predicates for reusable checks|e.g., `isString(value): value is string`|
|🧱 Combine narrowing techniques|TypeScript can chain them (e.g., `typeof` + `in`)|

---

## ⚠️ Common Pitfalls

|Pitfall|Example|Fix|
|---|---|---|
|Forgetting `strictNullChecks`|TS won’t narrow `string|null` properly|
|Using unsafe assertions|`(x as number).toFixed()`|Use `typeof` guard instead|
|Missing `default` in switch|TS can’t detect exhaustive checks|Add `default` or `never` case|

---

## 🧠 Summary

|Concept|Description|Example|
|---|---|---|
|**Narrowing**|Refine union or broad type into a specific one|`if (typeof x === "string")`|
|**Type Guards**|Logical checks that narrow types|`typeof`, `instanceof`, `in`|
|**Discriminated Union**|Union with a shared tag field|`{ kind: "circle" }`|
|**Type Predicate**|Custom narrowing function|`animal is Cat`|
|**Control Flow Analysis**|Tracks type changes across code|`if (value) { ... }`|

---

## ✅ Quick Recap Example

```ts
type Animal =
  | { kind: "dog"; bark: () => void }
  | { kind: "cat"; meow: () => void };

function makeSound(a: Animal) {
  if (a.kind === "dog") a.bark();
  else a.meow();
}
```

TypeScript automatically narrows based on the `kind` property — clean, safe, and predictable.