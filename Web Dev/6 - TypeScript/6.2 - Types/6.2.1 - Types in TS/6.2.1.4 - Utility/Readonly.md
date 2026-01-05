---
tags: 
 - typescript
 - type
 - utility
---

`Readonly<T>` is a **[[Mapped|mapped utility type]]** that converts **all properties** of a given type `T` into `readonly` properties.

```ts
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

It does **not** change the values themselves—only how TypeScript allows you to interact with them.

---

## 🧩 What `Readonly<T>` Does

```ts
type User = {
  id: string;
  name: string;
};

type ReadonlyUser = Readonly<User>;
```

Resulting type:

```ts
{
  readonly id: string;
  readonly name: string;
}
```

Any attempt to reassign properties will produce a **compile-time error**.

---

## 🛑 Mutation Is Disallowed

```ts
const user: Readonly<User> = {
  id: "u1",
  name: "Long",
};

user.name = "Pham"; // ❌ Error
user.id = "u2";     // ❌ Error
```

This ensures the object is treated as **immutable after creation**.

---

## ⚠️ Shallow Immutability Only

`Readonly<T>` is **shallow**, not deep.

```ts
type State = {
  config: {
    theme: string;
  };
};

const s: Readonly<State> = {
  config: { theme: "dark" },
};

s.config.theme = "light"; // ✅ allowed
```

Only the **top-level properties** are readonly.

---

## ⚛️ Common React Use Case

React props are conceptually immutable. `Readonly<T>` makes this explicit.

```ts
type Props = {
  title: string;
  count: number;
};

function Header(props: Readonly<Props>) {
  // props.count++; ❌ Error
  return <h1>{props.title}</h1>;
}
```

This prevents accidental mutation of props inside components.

---

## 🧠 Relationship to `as const`

- `Readonly<T>` works on **types**
    
- `as const` works on **values**
    

```ts
const config = {
  mode: "dark",
  retry: 3,
} as const;
```

Equivalent to:

```ts
Readonly<{
  mode: string;
  retry: number;
}>
```

…but with **literal types preserved**.

---

## 📦 `Readonly<T>` vs Manual `readonly`

### Manual

```ts
type User = {
  readonly id: string;
  readonly name: string;
};
```

### Utility Type

```ts
type ReadonlyUser = Readonly<User>;
```

✔ Cleaner  
✔ Easier to refactor  
✔ Ideal for API boundaries

---

## ✅ When to Use `Readonly<T>`

- Function parameters you must not mutate
    
- React props
    
- Public APIs
    
- State snapshots
    
- Defensive typing against accidental writes
    

---

## 🧾 Summary

- `Readonly<T>` makes **all properties of `T` immutable**
    
- It is **shallow**
    
- It is enforced **at compile time only**
    
- It improves correctness and intent clarity
    
