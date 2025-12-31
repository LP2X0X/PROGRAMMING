---
tags: 
 - typescript
 - type
 - utility
---

## 🧱 `Required<T>` (TypeScript Utility Type)

### Category

- **Utility type**
    
- **Mapped type**
    
- **Non-distributive**
    

---

## What `Required<T>` does

```ts
Required<T>
```

Makes **all properties of `T` required** (removes `?`).

---

## Basic example

```ts
type User = {
  id?: number;
  name?: string;
  active?: boolean;
};

type StrictUser = Required<User>;
```

Result:

```ts
{
  id: number;
  name: string;
  active: boolean;
}
```

---

## How it works internally

```ts
type Required<T> = {
  [K in keyof T]-?: T[K];
};
```

Key parts:

- `keyof T` → key union
    
- `in` → mapped iteration
    
- `-?` → **removes optional modifier**
    
- `T[K]` → property type
    

---

## `-?` modifier (important)

Mapped types support **modifier removal**:

|Modifier|Meaning|
|---|---|
|`?`|Add optional|
|`-?`|Remove optional|
|`readonly`|Add readonly|
|`-readonly`|Remove readonly|

---

## `Required` is **not distributive**

```ts
type A = { a?: number };
type B = { b?: string };

type X = Required<A | B>;
```

Result:

```ts
{
  a: number;
  b: string;
}
```

Not:

```ts
Required<A> | Required<B>
```

---

## When to use `Required`

- Normalize optional config objects
    
- Enforce completeness after validation
    
- API response normalization
    
- Internal invariants
    

---

## Common pitfall

```ts
type T = {
  x?: number;
};

type R = Required<T>;
```

This **does not remove `undefined`** if it exists explicitly:

```ts
x?: number | undefined  // becomes
x: number | undefined
```

Optionality ≠ undefined removal.

````ad-warning
This only effect the "shallow" properties, not nested ones.
```ts
type Coordinates = {
  x?: number;
  y?: number;
  a?: {
	 b?: number; // If Required apply to Coordinates, b will still not required.
  };
};
```
````

---

## Related utility types

| Utility       | Purpose        |
| ------------- | -------------- |
| `Partial<T>`  | Make optional  |
| `Required<T>` | Make required  |
| `Readonly<T>` | Make immutable |
| `Pick<T, K>`  | Select keys    |
| `Omit<T, K>`  | Remove keys    |

---

## Mental model

> **`Required<T>` restores structural obligation, not value certainty.**

---

## One-line takeaway

> **`Required<T>` removes the optional (`?`) modifier from all properties using a mapped type.**