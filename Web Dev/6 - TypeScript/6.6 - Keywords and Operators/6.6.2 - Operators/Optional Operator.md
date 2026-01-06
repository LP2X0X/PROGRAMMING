---
tags: 
 - typescript
 - operator
---

## 1️⃣ What `?` means in TypeScript

> The `?` operator marks a property or parameter as **optional**.

It means:

- The value **may be missing**
    
- The type is **not guaranteed to exist**
    

---

## 2️⃣ Optional object properties

```ts
type User = {
  id: number;
  name?: string;
};
```

Valid values:

```ts
{ id: 1 }
{ id: 1, name: 'Alice' }
```

Invalid:

```ts
{ name: 'Alice' } // missing required `id`
```

---

## 3️⃣ What type does `?` produce?

Under `strictNullChecks` (default in modern TS):

```ts
name?: string
```

Is equivalent to:

```ts
name: string | undefined
```

But with an important semantic difference (see section 6).

---

## 4️⃣ Optional function parameters

```ts
function greet(name?: string) {
  console.log(name?.toUpperCase());
}
```

Equivalent to:

```ts
function greet(name: string | undefined) {}
```

Call sites:

```ts
greet();
greet('Bob');
```

---

## 5️⃣ Optional methods

```ts
type Service = {
  connect?(): void;
};
```

Usage:

```ts
service.connect?.();
```

---

## 6️⃣ Optional vs `| undefined` (important difference)

```ts
type A = { x?: number };
type B = { x: number | undefined };
```

### Key difference:

```ts
const a: A = {};              // ✅ OK
const b: B = {};              // ❌ Error (property missing)
```

> `?` allows the property to be **absent**  
> `| undefined` requires the property to **exist**, possibly undefined

---

## 7️⃣ Optional chaining (`?.`) is related but different

```ts
user.profile?.email
```

- Safe access
    
- Prevents runtime errors
    
- Does **not** make the property optional in the type
    

---

## 8️⃣ Optional in interfaces vs types

Works identically:

```ts
interface Config {
  debug?: boolean;
}
```

```ts
type Config = {
  debug?: boolean;
};
```

---

## 9️⃣ Optional with destructuring

```ts
function init({ port = 3000 }: { port?: number }) {}
```

- Optional allows omission
    
- Default value handles `undefined`
    

---

## 1️⃣0️⃣ Optional + readonly

```ts
type Settings = {
  readonly theme?: string;
};
```

---

## Mental model

> **`?` means “this may not exist at all.”**

---

## Final takeaway

- `?` marks properties or parameters as optional
    
- It implies `| undefined` **plus** “may be missing”
    
- Optional is about **object shape**, not just value type
    
- Very common in API types, config objects, and React props