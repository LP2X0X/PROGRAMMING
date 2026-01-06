---
tags: 
 - typescript
 - operator
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
    

````ad-note
typeof can also be use to "extract" the type from an object:
```ts
const configurations = {
  development: {
    apiBaseUrl: "http://localhost:8080",
    timeout: 5000,
  },
  production: {
    apiBaseUrl: "https://api.example.com",
    timeout: 10000,
  },
  staging: {
    apiBaseUrl: "https://staging.example.com",
    timeout: 8000,
  },
};

type Configurations = typeof configurations;

/*
type Configurations = {
    development: {
        apiBaseUrl: string;
        timeout: number;
    };
    production: {
        apiBaseUrl: string;
        timeout: number;
    };
    staging: {
        apiBaseUrl: string;
        timeout: number;
    };
}
*/
```
````

---

### Rule of Thumb

> Use `typeof` for primitives, `instanceof` or custom guards for objects.