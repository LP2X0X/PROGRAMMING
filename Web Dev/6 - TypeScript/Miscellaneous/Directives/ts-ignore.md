---
tags: 
 - typeScript
 - directive
---

## 🚫 What `@ts-ignore` Is

`@ts-ignore` is a **TypeScript directive comment** that tells the compiler to **silence the next line’s error**, regardless of what the error is.

```ts
// @ts-ignore
someInvalidCode();
```

It suppresses **all TypeScript errors** on the **very next line only**.

---

## 🧠 How It Works

- Evaluated **at compile time**
    
- Applies to **one line**
    
- Completely bypasses TypeScript’s type checking for that line
    
- The code is still emitted to JavaScript unchanged
    

TypeScript essentially says:

> “I see a problem here, but I will pretend it does not exist.”

---

## ⚠️ What `@ts-ignore` Does NOT Do

- ❌ It does **not** fix the error
    
- ❌ It does **not** add runtime safety
    
- ❌ It does **not** narrow types
    
- ❌ It does **not** affect JavaScript behavior
    

Runtime errors are still possible.

---

## 📉 Why `@ts-ignore` Is Dangerous

```ts
// @ts-ignore
user.age.toFixed();
```

If `age` is actually `undefined`, your app will crash at runtime.

TypeScript’s protection is completely disabled for that line.

---

## 🆚 `@ts-ignore` vs `@ts-expect-error`

### `@ts-ignore`

```ts
// @ts-ignore
foo();
```

- Suppresses error **even if there is no error**
    
- Can hide future problems
    
- TypeScript does **not warn you**
    

---

### `@ts-expect-error` ✅ (Preferred)

```ts
// @ts-expect-error
foo();
```

- Suppresses error **only if an error exists**
    
- TypeScript **fails the build** if the error disappears
    
- Safer for maintenance
    

➡️ Use this when you **intentionally expect** an error.

---

## ✅ When `@ts-ignore` Is Acceptable

Use it **only as a last resort**, for example:

- Third-party libraries with broken typings
    
- Temporary migration code
    
- Legacy JavaScript interop
    
- Known compiler bugs
    

Even then, it should be **documented**.

---

## ❌ When You Should NOT Use It

- To avoid learning TypeScript
    
- To skip proper typing
    
- In shared or library code
    
- As a permanent solution
    

If you need many `@ts-ignore`s, your types are broken.

---

## 🧠 Better Alternatives (Almost Always)

### 1️⃣ Fix the type

```ts
(value as string).toUpperCase();
```

---

### 2️⃣ Narrow properly

```ts
if (typeof value === "string") {
  value.toUpperCase();
}
```

---

### 3️⃣ Use assertion functions

```ts
function assertString(x: unknown): asserts x is string {
  if (typeof x !== "string") throw new Error();
}
```

---

### 4️⃣ Use `unknown` instead of `any`

```ts
function parse(x: unknown) {
  if (typeof x === "string") {
    return x.length;
  }
}
```

---

## 🧠 Mental Model

> `@ts-ignore` is a **mute button**, not a fix.

---

## 📌 Best Practice Summary

- 🚫 Avoid `@ts-ignore`
    
- ✅ Prefer `@ts-expect-error`
    
- 🛠 Fix types instead of silencing them
    
- 📄 Document every suppression
    

---

## 🧠 One-Sentence Summary

> `@ts-ignore` forcibly disables TypeScript checking for a line and should be used only as a temporary, well-documented last resort.