---
tags: 
 - typescript
 - operator
---

## `extends` in TypeScript 

### Category

- **Type constraint operator**
    
- **Conditional type operator**
    
- **Structural typing keyword**
    

> `extends` does **not** always mean inheritance.

---

## 1️⃣ `extends` as a **generic constraint**

```ts
type Foo<T extends U> = ...
```

Meaning:

> **“`T` must be assignable to `U`.”**

Example:

```ts
type Keys<T extends PropertyKey> = T;
```

Valid:

```ts
Keys<string>
Keys<number>
Keys<symbol>
```

Invalid:

```ts
Keys<boolean> // ❌
```

This is how TypeScript enforces **allowed input types**.

---

## 2️⃣ `extends` in **conditional types**

```ts
T extends U ? X : Y
```

Meaning:

> **“If `T` is assignable to `U`, choose `X`; otherwise choose `Y`.”**

Example:

```ts
type IsString<T> = T extends string ? true : false;
```

---

## 3️⃣ `extends` causes **distribution** (important)

When `T` is a union:

```ts
T extends U ? X : Y
```

Becomes:

```ts
(A extends U ? X : Y) |
(B extends U ? X : Y)
```

This is why your **DistributedOmit** works.

---

## 4️⃣ `extends` in interfaces / classes (structural)

```ts
interface A {
  x: number;
}

interface B extends A {
  y: number;
}
```

Meaning:

> **“B must contain everything A has (and possibly more).”**

This is **structural**, not nominal inheritance.

---

## What `extends` is NOT

- ❌ Not classical inheritance (like Java)
    
- ❌ Not runtime behavior
    
- ❌ Not subtype creation by default
    

It is purely **compile-time checking**.

---

## Mental model (use this)

> **`extends` = “is assignable to”**

Not:

- “inherits from”
    
- “is a subclass of”
    

---

## Why TypeScript reuses the word `extends`

Because all these ideas reduce to **assignability**:

```ts
string extends PropertyKey // true
```

---

## One-line takeaway

> **In TypeScript, `extends` means “must be assignable to”, and it is used for constraints, conditionals, and structural extension.**