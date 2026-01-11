---
tags: 
 - typeScript
 - directive
---

## 🚫 What `@ts-nocheck` Is

`// @ts-nocheck` is a **TypeScript directive comment** that **disables all type checking for an entire file**.

```ts
// @ts-nocheck

const x: number = "hello"; // no error
foo.bar.baz();             // no error
```

Once present, TypeScript **skips the file completely**.

---

## 🧠 How It Works

- Evaluated **at compile time**
    
- Applies to the **entire file**
    
- Suppresses **every TypeScript error**
    
- JavaScript is still emitted normally
    

TypeScript treats the file as:

> “This file is untyped. Trust the developer.”

---

## ⚠️ What `@ts-nocheck` Does NOT Do

- ❌ Does not fix incorrect types
    
- ❌ Does not add runtime safety
    
- ❌ Does not narrow types
    
- ❌ Does not convert JS → TS safely
    

It only **turns the checker off**.

---

## 🆚 `@ts-nocheck` vs `@ts-ignore`

|Directive|Scope|Risk|
|---|---|---|
|`@ts-ignore`|One line|Medium|
|`@ts-nocheck`|Entire file|Very high|

`@ts-nocheck` is **far more dangerous**.

---

## 🆚 `@ts-nocheck` vs `@ts-check`

### Enable checking in JS files

```js
// @ts-check
```

### Disable checking

```js
// @ts-nocheck
```

These are commonly used in **`.js` files**, not `.ts`.

---

## 🧠 When `@ts-nocheck` Is Acceptable

Use it **temporarily** in very limited cases:

- Migrating a large legacy JS file to TS
    
- Third-party generated code
    
- Code that will be deleted soon
    
- Emergency unblock during refactor
    

Even then:

- Leave a **comment explaining why**
    
- Create a **task to remove it**
    

---

## ❌ When You Should NOT Use It

- In new TypeScript files
    
- In shared or library code
    
- To silence confusing errors
    
- As a permanent solution
    

If you need `@ts-nocheck`, your typing strategy is broken.

---

## 🛠 Better Alternatives

### 1️⃣ Gradual typing with `unknown`

```ts
function handle(x: unknown) {
  if (typeof x === "string") {
    console.log(x.toUpperCase());
  }
}
```

---

### 2️⃣ Fix specific lines only

```ts
// @ts-expect-error
legacyApi();
```

---

### 3️⃣ Split legacy code

Move unsafe code into a **small isolated file**.

---

### 4️⃣ Use `any` strategically (still risky)

```ts
const data: any = getLegacyData();
```

Better than nuking the whole file.

---

## 🧠 Mental Model

> `@ts-nocheck` is a **global kill switch** for TypeScript.

---

## 📌 Best Practice Summary

- 🚫 Avoid `@ts-nocheck`
    
- ⚠️ Use only as a short-term escape hatch
    
- 📝 Always document why
    
- 🛠 Remove it as soon as possible
    

---

## 🧠 One-Sentence Summary

> `@ts-nocheck` disables TypeScript checking for an entire file and should be used only as a temporary migration or emergency measure.