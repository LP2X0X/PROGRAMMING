---
tags: 
 - typescript
 - term
---

In TypeScript, an **Index Signature** is used when you don’t know the exact names of an object's properties in advance, but you do know the **shape** of the values and the **type** of the keys.

Think of it as a way to define a "dictionary" or a "map."

---

### 1. Basic Syntax

An index signature is written using square brackets in an interface or type alias.

```ts
interface UserCache {
  [key: string]: string; 
}

const cache: UserCache = {
  active: "yes",
  expired: "no",
  // You can add any string key you want!
};
```

- `key`: This is just a placeholder name (you can call it `id`, `name`, etc.).
    
- `string`: The type of the key (must be `string`, `number`, `symbol`, or a template literal).
    
- `string` (after the colon): The type of the value.
    

---

### 2. Key Restrictions

You can only use specific types for the index key:

- **`string`**: The most common (covers all object keys).
    
- **`number`**: Used for array-like objects.
    
- **`symbol`**: For unique property keys.
    
- **Template Literals**: (e.g., `` `data-${string}` ``).
    

```ad-note
You can use the type [[PropertyKey]] type for this.
```

---

### 3. Mixing Predefined and Index Properties

You can mix explicit properties with an index signature, but there is a **strict rule**: All explicit properties must match the return type of the index signature.

```ts
interface Metadata {
  id: number;            // ❌ Error!
  [key: string]: string; // Because 'id' must be a string to satisfy this
}

interface ValidMetadata {
  id: string;            // ✅ Works
  [key: string]: string; 
}
```

**Why?** Because `metadata["id"]` is accessed via a string key, so it must return whatever the index signature promises (in this case, a `string`).

```ts
interface RequiredMetadata {
  id: string;            
}

interface Metadata extends RequiredMetadata {
  [key: string]: string; 
}
```

---

### 4. The `readonly` Index Signature

You can make a dictionary "read-only" to prevent any values from being changed after the object is created.

```ts
interface ReadOnlyDict {
  readonly [key: string]: number;
}

const scores: ReadOnlyDict = { math: 100 };
scores.math = 90; // ❌ Error: Index signature is read-only.
```

---

### 5. Index Signature vs. `Record<K, T>`

In modern TypeScript, you will often see `Record<string, number>` instead of an index signature. They are mostly identical, but `Record` is more concise.

| **Feature**    | **Index Signature**            | **Record<string, number>**      |
| -------------- | ------------------------------ | ------------------------------- |
| **Syntax**     | `{[k: string]: number}`        | `Record<string, number>`        |
| **Usage**      | Better for interfaces/classes. | Better for quick type aliases.  |
| **Complexity** | Supports template literals.    | Supports unions (`'a' \| 'b'`). |

---

### 6. The "Missing Key" Danger

One major downside of index signatures is that TypeScript assumes every key you access exists.

```ts
interface Inventory {
  [item: string]: number;
}

const shop: Inventory = {};
console.log(shop.apples.toFixed(0)); // ❌ Runtime Crash! 
// TS thinks 'shop.apples' is a number, but it's actually undefined.
```

**The Fix:** Always include `undefined` in the value type if you aren't sure the key exists.

```ts
interface SafeInventory {
  [item: string]: number | undefined;
}
```

---

### Summary Checklist

- Use it when the keys are **dynamic** (like an API response where keys are IDs).
    
- Remember that all **fixed properties** must match the index signature type.
    
- Be careful with **undefined** values; TS won't warn you by default if a key is missing.
    