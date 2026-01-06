---
tags: 
 - typescript
 - feature
---

In TypeScript, a **namespace** is a way to **group related values, functions, and types under a single name** to avoid naming collisions.

It is a **TypeScript-only** construct and is largely considered **legacy** in modern code.

---

## Basic Example

```ts
namespace MathUtils {
  export function add(a: number, b: number) {
    return a + b;
  }

  export const PI = 3.14;
}

MathUtils.add(1, 2);
```

```ts
namespace GeometryUtils {
  export namespace Circle {
    export function calculateArea(radius: number) {
      // implementation
    }

    export function calculateCircumference(radius: number) {
      // implementation
    }
  }

  export namespace Rectangle {
    export interface Rectangle {
      width: number;
      height: number;
    }
    export function calculateArea(rect: Rectangle) {
      // implementation
    }

    export function calculatePerimeter(rect: Rectangle) {
      // implementation
    }
  }
}

// Can be used as values...
GeometryUtils.Circle.calculateArea(10);

// ...or as types
const rect: GeometryUtils.Rectangle.Rectangle = {
  width: 10,
  height: 20,
};
```

Without `export`, members are private to the namespace.

---

## What It Compiles To

Namespaces have **runtime semantics**.

The above code emits JavaScript similar to:

```js
var MathUtils;
(function (MathUtils) {
  MathUtils.add = function (a, b) {
    return a + b;
  };
  MathUtils.PI = 3.14;
})(MathUtils || (MathUtils = {}));
```

This is why namespaces are **not erasable**.

---

## Why They Exist

Namespaces predate ES modules and were used to:

- Organize large codebases
    
- Avoid global name collisions
    
- Simulate modules before `import` / `export`
    

---

## Why They Are Discouraged Today

In modern TypeScript:

- **ES modules replace namespaces**
    
- Modules are statically analyzable
    
- Tooling, bundlers, and Node work better with modules
    

Instead of:

```ts
namespace Utils { ... }
```

Prefer:

```ts
export function helper() {}
```

---

## Namespaces vs Modules (Key Difference)

| Aspect            | Namespace   | ES Module           |
| ----------------- | ----------- | ------------------- |
| Runtime code      | Yes         | Yes                 |
| Syntax            | `namespace` | `import` / `export` |
| File-based        | No          | Yes                 | 
| Recommended today | ❌          | ✅                  |

---

## Important Gotchas

### Not supported in Node `--experimental-strip-types`

Namespaces generate runtime objects, so they **cannot be stripped**.

### Declaration merging

Namespaces can merge with:

- Other namespaces
    
- Classes
    
- Functions
    

```ts
class User {}
namespace User {
  export const role = "admin";
}
```

This works but increases complexity.

```ad-note
All or none must be exported.
```

```ad-note
Interface merging work normally in this case.
```

---

## When (If Ever) to Use Them

Use namespaces only when:

- Working with **global scripts** (no modules)
    
- Writing **.d.ts declaration files**
    
- Maintaining legacy TypeScript code
    

---

## One-Line Summary

> **Namespaces are a pre-module way to organize code, generate runtime objects, and are mostly replaced by ES modules in modern TypeScript.**