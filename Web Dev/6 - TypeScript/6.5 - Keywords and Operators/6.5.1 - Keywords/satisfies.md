---
tags: 
 - typeScript
 - operator
---

## ✅ What `satisfies` Is

`satisfies` is a **TypeScript keyword** that checks whether a value **conforms to a type** **without changing the [[Type Inference|inferred type]] of the value**.

```ts
const user = {
  id: 1,
  name: "Alice",
} satisfies User;
```

It is a **type constraint**, not a type assertion.

---

## 🧠 The Core Problem It Solves

Before `satisfies`, you had to choose between:

### ❌ Losing inference

```ts
const user: User = {
  id: 1,
  name: "Alice",
};
```

Now `user` is exactly `User` — extra precision is lost.

---

### ❌ Unsafe assertion

```ts
const user = {
  id: 1,
  name: "Alice",
} as User;
```

This **forces TypeScript to trust you**, even if the object is wrong.

````ad-note
Another way to think of it is that with [[Type Inference|type inference]] on a const object, you can access the properties of it without having to be precise with type checking.
```ts
type Color =
  | string
  | {
      r: number;
      g: number;
      b: number;
    };

const config = {
  foreground: { r: 255, g: 255, b: 255 },
  background: { r: 0, g: 0, b: 0 },
  border: "transparent",
};

// No problem here
config.border.toUpperCase();
console.log(config.foreground.r);
```

But the type of property might not actually be the one you expect it to be:
```ts
type Color =
  | string
  | {
      r: number;
      g: number;
      b: number;
    };

// Imagine we need the values of background to be of type Color
const config = {
  foreground: { r: 255, g: 255, b: 255 },
  background: { r: 0, g: 0 }, // Missing `b` here, no warning or error
  border: "transparent",
};
```

We can annotate the type of config to ensure the type of it, but when trying to access stuff inside it, we need to narrowing stuff down:
```ts
type Color =
  | string
  | {
      r: number;
      g: number;
      b: number;
    };

const config : Record<string, Color> = {
  foreground: { r: 255, g: 255, b: 255 },
  background: { r: 0, g: 0 }, // Type '{ r: number; g: number; }' is not assignable to type 'Color'. 
  border: "transparent",
};

// Must narrow down types before use
config.border.toUpperCase();
console.log(config.foreground.r);
```

**Type inference makes the type more precise while type annotation makes it more strict. Then how can we get the best of both worlds?**
````

---

### ✅ `satisfies` (Best of both worlds)

```ts
const user = {
  id: 1,
  name: "Alice",
} satisfies User;
```

- TypeScript **verifies correctness of types**
    
- **Keeps the exact inferred type**
    
- No unsafe casting
    

---

## 🎯 What `satisfies` Actually Does

`satisfies` answers one question:

> “Does this value conform to this type?”

If yes → OK  
If no → **compile-time error**

But it **does not replace the value’s type**.

---

## 🔍 Key Difference vs `:` (Type Annotation)

```ts
type Config = {
  mode: "dev" | "prod";
};

const config1: Config = {
  mode: "dev",
};
// config1.mode is "dev" | "prod"

const config2 = {
  mode: "dev",
} satisfies Config;
// config2.mode is "dev"
```

`satisfies` preserves **literal types**.

---

## 🧩 Common Real-World Use Cases

---

## ⚛️ Configuration Objects

```ts
type AppConfig = {
  apiUrl: string;
  retry: number;
};

const config = {
  apiUrl: "https://api.example.com",
  retry: 3,
  debug: true,
} satisfies AppConfig;
```

❌ Error: `debug` is not allowed  
✅ Exact checking without losing inference

---

## 🧪 Maps and Dictionaries

```ts
type Roles = "admin" | "user";

const permissions = {
  admin: ["read", "write"],
  user: ["read"],
} satisfies Record<Roles, string[]>;
```

Prevents missing keys or typos.

---

## 🧠 Function Tables

```ts
type Handlers = {
  click: () => void;
  submit: () => void;
};

const handlers = {
  click: () => {},
  submit: () => {},
} satisfies Handlers;
```

Ensures all handlers exist, keeps exact function types.

---

## ⚠️ `satisfies` vs `as`

| Feature             | `satisfies` | `as`    |
| ------------------- | ----------- | ------- |
| Type safety         | ✅ Yes      | ❌ No   |
| Preserves inference | ✅ Yes      | ❌ No   |
| Runtime impact      | ❌ None     | ❌ None |
| Can hide errors     | ❌ No       | ✅ Yes  |

Rule of thumb:

> **Use `satisfies` to verify, `as` only when you must override.**

---

## 🧠 `satisfies` Does NOT Narrow Variables

```ts
const value = "hello" satisfies string;
```

This is pointless — `satisfies` is for **structure**, not narrowing.

---

## 🧠 Mental Model

> `satisfies` is a **compile-time contract check**, not a cast.

---

## 📌 When You Should Use `satisfies`

- Configuration objects
    
- Lookup tables
    
- Enforcing object shape without losing literals
    
- Preventing silent mistakes
    
- Replacing unsafe `as` casts
    

---

## 🚫 When NOT to Use It

- Simple variables
    
- Runtime validation
    
- Narrowing control flow
    
- Replacing `typeof` or `instanceof`
    

---

## 🧠 One-Sentence Summary

> `satisfies` verifies that a value conforms to a type **without changing its inferred type**, making it safer and more precise than `as`.

----

The `satisfies` operator (introduced in TypeScript 4.9) is a keyword that allows you to **validate** that a value matches a specific type, without **changing** (or "widening") the inferred type of that value.

In simple terms: It checks if your code follows the rules, but it doesn't make your code "forget" the specific details of the value you wrote.

---

## 1. The Problem it Solves (The "Colon" Issue)

Before `satisfies`, we used the colon (`:`) to annotate types. However, using a colon tells TypeScript: _"Forget exactly what this value is, and treat it broadly as this Type."_ This is called **Type Widening**.

### The "Old" Way (Data Loss)

Imagine a configuration where colors can be a **string** (hex) or an **RGB object**.

```ts
type Color = string | { r: number; g: number; b: number };
type Palette = Record<string, Color>;

// ❌ Using Colon (: Palette)
const myColors: Palette = {
    red: "#FF0000",
    green: { r: 0, g: 255, b: 0 }
};

// ERROR! TypeScript thinks 'red' MIGHT be an object now.
// It has lost the information that 'red' is specifically a string.
myColors.red.toUpperCase(); 
// ^ Property 'toUpperCase' does not exist on type 'Color'.
```

Even though _we_ can see `red` is a string, TypeScript "widened" it to match the broader `Color` definition (`string | object`), so it forces you to write safe checks (like `if (typeof ...)`).

---

## 2. The `satisfies` Solution

Using `satisfies` checks that `myColors` conforms to the `Palette` structure, but preserves the **exact** types of the values you provided.

```ts
// ✅ Using satisfies
const myColors = {
    red: "#FF0000",
    green: { r: 0, g: 255, b: 0 }
} satisfies Palette;

// WORKS! TypeScript knows 'red' is exactly a string here.
myColors.red.toUpperCase(); 

// WORKS! TypeScript knows 'green' is exactly an object.
myColors.green.r.toFixed();

// VALIDATION: This still errors if you make a mistake
const badColors = {
    blue: true // Error: 'boolean' is not assignable to 'Color'
} satisfies Palette;
```

---

## 3. Comparison Table

|**Feature**|**const x: Type = {} (Annotation)**|**const x = {} satisfies Type**|
| ---                 | ---                                                     | ---                                                           |
| ------------------- | ------------------------------------------------------- | ------------------------------------------------------------- |
| **Primary Goal**    | Ensure the variable **IS** the Type.                    | Ensure the value **MATCHES** the Type.                        |
| **Inference**       | **Widening**. The value is generalized to the Type.     | **Narrow/Exact**. The value keeps its specific shape.         |
| **Strictness**      | Enforces the contract.                                  | Enforces the contract.                                        |
| **Property Access** | Can only access properties defined in the generic Type. | Can access properties specific to the actual object provided. |

---

## 4. Best Use Case: `satisfies` + `as const`

The most powerful combination is using `satisfies` along with `as const`. This creates a deeply immutable object that is validated against a type but retains literal values.

This is extremely common in routing or theme configurations.

```ts
type Routes = Record<string, { path: string; protected: boolean }>;

// Using both ensures:
// 1. keys match the Routes shape (satisfies)
// 2. keys are strictly "literal" strings, not just generic strings (as const)
const AppRoutes = {
    home: { path: "/", protected: false },
    dashboard: { path: "/admin", protected: true }
} as const satisfies Routes;

// TypeScript knows 'home' path is exactly "/" (literal), not just 'string'
// This is great for autocomplete elsewhere in your app.
```

---

## 5. Summary

- Use **`: Type`** when you want to restrict a variable to a known interface and don't care about the specific value currently inside it.
    
- Use **`satisfies Type`** when you want to ensure a value is "correct" (validates against the type) but you want to keep using its specific properties without TypeScript "forgetting" them.
    