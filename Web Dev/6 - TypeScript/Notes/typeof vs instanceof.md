---
tags: 
 - typescript
 - distinguish
 - note
---

In TypeScript (and JavaScript), `typeof` and `instanceof` are both operators used to determine the type of a value, but they work in fundamentally different ways and serve different purposes.

Here is the quick breakdown of the differences:

| **Feature**       | **typeof**                         | **instanceof**                      |
| ----------------- | ---------------------------------- | ----------------------------------- |
| **Operands**      | Unary (e.g., `typeof x`)           | Binary (e.g., `x instanceof Class`) |
| **Returns**       | A **string** representing the type | A **boolean** (`true` / `false`)    |
| **Primary Use**   | Checking **Primitive** types       | Checking **Class/Object** types     |
| **Mechanism**     | Checks the data type tag           | Checks the **Prototype Chain**      |
| **TS Type Guard** | Yes (for primitives)               | Yes (for classes)                   |

---

### 1. `typeof`

The `typeof` operator returns a string string indicating the type of the unevaluated operand. It is the go-to operator for checking **primitives**.

Return values:

It returns one of the following strings: 'string', 'number', 'boolean', 'undefined', 'object', 'function', 'symbol', or 'bigint'.

#### **When to use it:**

- Checking if a variable is a string, number, or boolean.
    
- Checking if a function is defined.
    
- Safely checking for `undefined`.
    

#### **TypeScript Type Guard Example:**

TypeScript is smart enough to "narrow" the type inside a `typeof` check.

```ts
function printId(id: string | number) {
  if (typeof id === "string") {
    // TypeScript knows 'id' is a string here
    console.log(id.toUpperCase());
  } else {
    // TypeScript knows 'id' is a number here
    console.log(id.toFixed(2));
  }
}
```

#### **The "Gotchas" (Limitations):**

`typeof` is notoriously bad at distinguishing between different types of objects.

- `typeof null` returns `'object'` (This is a historical JavaScript bug).
    
- `typeof []` (Array) returns `'object'`.
    
- `typeof new Date()` returns `'object'`.
    

---

### 2. `instanceof`

The `instanceof` operator tests to see if the `prototype` property of a constructor appears anywhere in the prototype chain of an object. It is the go-to operator for checking **Classes** and specific **Object types**.

#### **When to use it:**

- Checking if an object is an instance of a specific **Class**.
    
- Checking for built-in complex objects like `Array`, `Date`, `Error`, or `RegExp`.
    

#### **TypeScript Type Guard Example:**

Just like with `typeof`, TypeScript treats `instanceof` as a type guard.

```ts
class Dog {
  bark() { console.log("Woof!"); }
}

class Cat {
  meow() { console.log("Meow!"); }
}

function handlePet(pet: Dog | Cat) {
  if (pet instanceof Dog) {
    // TypeScript knows 'pet' is a Dog here
    pet.bark();
  } else {
    // TypeScript knows 'pet' is a Cat here
    pet.meow();
  }
}
```

#### **The "Gotchas" (Limitations):**

- **Primitives:** `instanceof` returns `false` for primitives, even if they have a wrapper object equivalent.
    
    ```ts
    "hello" instanceof String; // false
    new String("hello") instanceof String; // true
    ```
    
- **Cross-window/iframe contexts:** If you have an array created in one iframe and check `instanceof Array` in another, it may return `false` because the `Array` constructors are different memory references.
    

---

### Summary Checklist: Which one do I use?

1. **Is it a number, string, boolean, or undefined?**
    
    - Use **`typeof`**.
        
    - _Example:_ `if (typeof x === 'string')`
        
2. **Is it an Array?**
    
    - Do **not** use `typeof` (it returns 'object').
        
    - Use `Array.isArray(x)` (Best practice) or `x instanceof Array`.
        
3. **Is it a null value?**
    
    - Do **not** use `typeof` (it returns 'object').
        
    - Use strict equality: `if (x === null)`.
        
4. **Is it a Class instance (e.g., `Date`, `Error`, or `MyCustomClass`)?**
    
    - Use **`instanceof`**.
        
    - _Example:_ `if (err instanceof Error)`
        
5. **Is it a TypeScript Interface?**
    
    - **Neither works.** Interfaces are erased at runtime. To check for interfaces, you must use a custom **User-Defined Type Guard** (a function that returns `arg is MyInterface`).
        