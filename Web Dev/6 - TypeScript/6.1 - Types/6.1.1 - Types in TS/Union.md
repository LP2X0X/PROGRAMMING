---
tags: 
 - typescript
 - union
 - type
---

### 🔹 Definition

A **union type** lets a variable hold **more than one possible type**.  
You define it using the `|` (pipe) operator.

```ts
let value: string | number;
value = "hello"; // ✅
value = 42;      // ✅
value = true;    // ❌ not allowed
```

It means:

> “`value` can be a string **or** a number.”

---

### 🔹 Why Union Types Exist

They give **flexibility** while maintaining **type safety** —  
instead of using `any`, which disables type checking, you tell TypeScript the _limited_ set of allowed types.

---

### 🔹 Common Use Cases

|Use case|Example|
|---|---|
|Handling multiple input types|`string|
|APIs returning different formats|`User|
|Optional values before initialization|`T|
|Narrowing types with conditions|`string|

---

### 🧠 Example: Type Narrowing

TypeScript will only allow an operation if it is valid for _every_ member of the union.
The solution is to _narrow_ the union with code, TypeScript requires **narrowing** before using members of a specific type.

```ts
function printId(id: string | number) {
  // ❌ Error: Property 'toUpperCase' does not exist on type 'string | number'
  // console.log(id.toUpperCase());

  if (typeof id === "string") {
    console.log(id.toUpperCase()); // ✅ Safe: now TS knows it's a string
  } else {
    console.log(id.toFixed(2)); // ✅ Safe: now TS knows it's a number
  }
}
```

---

### 🔹 Union with Literal Types

You can combine literal types for limited, allowed values:

```ts
type Direction = "up" | "down" | "left" | "right";

let move: Direction;
move = "up";     // ✅
move = "forward"; // ❌ not assignable
```

This is useful for enums or configuration options.

---

### 🔹 Union of Object Types

You can also form unions of **object shapes**:

```ts
type Cat = { meow: () => void };
type Dog = { bark: () => void };

function makeSound(animal: Cat | Dog) {
  // ❌ Property 'bark' does not exist on type 'Cat | Dog'
  // animal.bark();

  if ("bark" in animal) {
    animal.bark(); // ✅ TS narrowed to Dog
  } else {
    animal.meow(); // ✅ TS narrowed to Cat
  }
}
```

---

### 🔹 Union with Null and Undefined

Common in optional values:

```ts
let name: string | null = null;

if (name !== null) {
  console.log(name.toUpperCase());
}
```

---

### 💡 Tips & Pitfalls

|Tip|Explanation|
|---|---|
|🔸 Use union instead of `any`|Keeps type safety|
|🔸 Always narrow before using|TS won’t allow mixed member access|
|🔸 Prefer literal unions for fixed choices|Easier validation and autocompletion|
|⚠️ Avoid overly broad unions|e.g. `string|
|⚠️ Watch for implicit unions|e.g. `let x; x = 1; x = "a";` infers `string|

---

### 🧭 Example: Combining with Type Aliases

```ts
type ID = string | number;

function getUser(id: ID) {
  console.log(`Fetching user ${id}`);
}
```

You can reuse unions across your code easily with type aliases.

---

### 🔹 Union vs Intersection

|Concept|Operator|Meaning|Example|
|---|---|---|---|
|**Union**|`|`|Can be one of multiple types|
|**Intersection**|`&`|Must satisfy all types|`{ a: number } & { b: string }`|

---

### 🧾 Summary

|Concept|Description|Example|
|---|---|---|
|Union|Value can be one of several types|`string \| number`|
|Type narrowing|TS checks actual type at runtime branch|`if (typeof x === "string")`|
|Literal union|Restrict value to fixed set|`"on" \| "off"`|
|Object union|Combine possible shapes|`{a:number} \| {b:string}`|

---

### ✅ Quick Reference Code

```ts
type Input = string | number | boolean;

function handle(input: Input) {
  if (typeof input === "string") console.log(input.toUpperCase());
  else if (typeof input === "number") console.log(input.toFixed(1));
  else console.log(input ? "TRUE" : "FALSE");
}
```

---

Would you like me to follow up with a **separate note for intersection types (`&`)** next — same format, so you can keep them side-by-side in Obsidian?