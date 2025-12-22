---
tags: 
 - ts
 - syntax
 - type
 - guard
---

`typeof` is a **built-in JavaScript operator** that TypeScript uses as a **[[Type Guard|type guard]]** to narrow primitive types at runtime.

---

### What `typeof` Returns

It always returns a **string**:

- `"string"`
    
- `"number"`
    
- `"boolean"`
    
- `"bigint"`
    
- `"symbol"`
    
- `"undefined"`
    
- `"object"`
    
- `"function"`
    

---

### Common Type Guard Usage

```ts
function format(value: string | number) {
  if (typeof value === "string") {
    // value is string here
    return value.toUpperCase();
  }

  // value is number here
  return value.toFixed(2);
}
```

TypeScript narrows the union based on the check.

---

### Valid `typeof` Checks in TypeScript

TypeScript only treats these as type guards:

```ts
typeof x === "string"
typeof x === "number"
typeof x === "boolean"
typeof x === "symbol"
typeof x === "bigint"
typeof x === "undefined"
typeof x === "function"
```

---

### Important Pitfall

```ts
typeof null === "object"; // true
```

Because of this:

- `typeof` is **not reliable for distinguishing objects**
    
- Use `instanceof` or property checks for objects
    

---

### When to Use `typeof`

- Narrowing **primitive unions**
    
- Safe runtime checks
    
- Simple, fast type discrimination
    

---

### Rule of Thumb

> Use `typeof` for primitives, `instanceof` or custom guards for objects.