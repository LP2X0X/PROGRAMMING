---
tags: 
 - typescript
 - type
 - check
---

## 🧠 What Is Type Assertion?

Type **assertion** tells TypeScript:

> “Trust me, I know what I’m doing — treat this value as having this specific type.”

It’s a way for you to **override TypeScript’s inferred type** when _you know more_ than the compiler.

💬 It’s similar to _type casting_ in other languages (like C++ or Java),  
but it **doesn’t change the runtime value** — it only changes the **type checking** at compile time.

---

## 🧩 Basic Syntax

TypeScript has **two equivalent syntaxes** for assertions:

### 1️⃣ Angle Bracket Syntax

```ts
let value: unknown = "hello";
let strLength = (<string>value).length;
```

### 2️⃣ `as` Syntax (preferred)

```ts
let value: unknown = "hello";
let strLength = (value as string).length;
```

✅ `as` syntax is recommended —  
it works in `.tsx` (React) files where angle brackets conflict with JSX.

---

## ⚙️ Example: Narrowing `unknown`

```ts
function getValue(): unknown {
  return "TypeScript";
}

let strLength = (getValue() as string).length;
```

Here:

- `getValue()` returns `unknown`
    
- We _assert_ that it’s a `string`, so `.length` becomes valid
    

---

## 🚨 Type Assertion vs Type Casting

| Feature           | Type Assertion                    | Type Casting                        |
| ----------------- | --------------------------------- | ----------------------------------- |
| Language Level    | TypeScript only                   | Runtime languages (C++, Java, etc.) |
| Effect at Runtime | **No effect**                     | Changes actual data type            |
| Purpose           | Hint TypeScript for type checking | Convert value at runtime            |

```ts
let num = "123" as unknown as number; // ✅ valid TS
// but still a string at runtime!
```

---

## 🧩 Assertion with DOM Elements

A common real-world case:

```ts
const input = document.querySelector("input") as HTMLInputElement;
console.log(input.value);
```

Without assertion, TypeScript sees the element as `Element | null` —  
you can’t access `.value`.  
By asserting, you tell the compiler it’s specifically an `<input>`.

⚠️ But make sure it **really exists**, or you’ll get a runtime error!

---

## 🧱 Double Assertions

You can assert _through_ `unknown` (or `any`) as an intermediate type:

```ts
let str = "hello";
let num = (str as unknown) as number;
```

❌ Avoid this unless absolutely necessary — it bypasses type safety completely.

---

## 🧠 Assertion vs Type Guards

| Feature     | Type Assertion                  | Type Guard                          |
| ----------- | ------------------------------- | ----------------------------------- |
| Type safety | Manual override                 | Checked at runtime                  |
| Example     | `(val as string).length`        | `if (typeof val === "string")`      |
| Use case    | You know the type ahead of time | You need to verify type dynamically |

💡 **Tip:** Prefer _type guards_ when possible — they’re safer.

---

## 🪄 Useful Use Cases

### ✅ 1. When narrowing DOM queries

```ts
const button = document.getElementById("save") as HTMLButtonElement;
button.disabled = true;
```

### ✅ 2. When working with `any` / `unknown`

```ts
const data: any = JSON.parse('{"name": "Alice"}');
console.log((data as { name: string }).name);
```

### ✅ 3. When migrating JS → TS gradually

```ts
let count = (<number><unknown>getValueFromLegacyCode());
```

### ✅ 4. Non-null Assertion (shortcut)

If you’re sure a value isn’t `null` or `undefined`:

```ts
const el = document.querySelector("#title")!;
el.textContent = "Hello";
```

The `!` (non-null assertion) is like saying:

> “Trust me, this value isn’t null.”

---

## 🧩 Common Pitfalls

| Pitfall                      | Example                                                       | Why it’s risky                                  |
| ---------------------------- | ------------------------------------------------------------- | ----------------------------------------------- |
| Overriding wrong type        | `("123" as number) + 1`                                       | No runtime conversion — still string!           |
| Ignoring `null` checks       | `(document.getElementById("id") as HTMLDivElement).innerText` | Crashes if `null`                               |
| Overusing assertions         | `(val as any).foo.bar.baz()`                                  | Loses all type safety                           |
| Using instead of type guards |                                                               | Type guards are safer, assertions just _assume_ |

---

## 🧭 Summary

| Concept            | Description                                         |
| ------------------ | --------------------------------------------------- |
| **Purpose**        | Tell TypeScript to treat a value as a specific type |
| **Runtime effect** | None — compile-time only                            |
| **Syntax**         | `value as Type` (recommended)                       |
| **Good for**       | DOM access, unknown/any, interop with JS            |
| **Avoid**          | Double assertion, unsafe overrides                  |

---

### ✅ Quick Example Recap

```ts
const el = document.querySelector("#name") as HTMLInputElement;
el.value = "TypeScript";

const value: unknown = "123";
const len = (value as string).length;

let x = "hi" as unknown as number; // ❌ unsafe
```