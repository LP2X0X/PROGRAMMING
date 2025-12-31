---
tags: 
 - typescript
 - type
 - special
---

`PropertyKey` is a built-in TypeScript type that represents **all valid JavaScript object property keys**.

```ts
type PropertyKey = string | number | symbol;
```

Any value used as an object key in JavaScript must belong to this union.

---

### Why it exists

JavaScript objects only allow **three kinds of keys**:

- `string`
    
- `number` (coerced to string at runtime)
    
- `symbol`
    

`PropertyKey` encodes this **runtime rule** at the type level, so TypeScript can correctly constrain APIs that work with object keys.

---

### Where it is used

`PropertyKey` is primarily used in **type-level constraints**, especially when defining generic object utilities.

Common contexts:

- `keyof`
    
- Index signatures
    
- Mapped types
    
- Generic dictionary patterns
    

```ts
type Dictionary<K extends PropertyKey, V> = {
  [P in K]: V;
};
```

---

### Relation to `keyof`

`keyof` always resolves to a subtype of `PropertyKey`.

```ts
type Keys = keyof { a: number; 1: string };
// type Keys = "a" | 1
```

This works because `"a" | 1` ⊆ `PropertyKey`.

---

### Why `number` is included

Although JavaScript converts numeric keys to strings internally:

```js
obj[1] === obj["1"]; // true
```

TypeScript preserves `number` in `PropertyKey` to:

- Match JavaScript syntax
    
- Improve typing of arrays, tuples, and numeric indexes
    

---

### Common mistake

❌ Treating `PropertyKey` as a utility or helper type  
It does **not transform** or reshape types — it **restricts** them.

---

### Mental model

> **`PropertyKey` = “Anything JavaScript allows between the brackets `obj[ ? ]`”**

---

### Category placement

- **Primary**: Special
    
- **Secondary**: Meta (type-level constraint)
    

---

### Related types

- `keyof`
    
- Index signatures
    
- `Record<K, V>`
    
- Mapped types
    
