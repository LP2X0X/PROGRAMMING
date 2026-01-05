---
tags: 
 - typescript
 - class
 - accessors
---

Getters and setters in TypeScript (also known as **accessors**) are special methods that allow you to intercept the reading and writing of a property. They look like properties to the outside world, but they execute logic behind the scenes.

--- 
### 1. Basic Syntax

```ts
class C {
  _length = 0;
  get length() {
    return this._length;
  }
  set length(value) {
    this._length = value;
  }
}
```

```ad-note
Note that a field-backed get/set pair with no extra logic is very rarely useful in JavaScript. It’s fine to expose public fields if you don’t need to add additional logic during the get/set operations.
```

```ad-note
Set accessor must have exactly one parameter and its type can be infer from the Get accessor.
```

- TypeScript has some special inference rules for accessors:
	- If get exists but no set, the property is automatically [[Web Dev/6 - TypeScript/6.6 - Keywords/6.5 - Type Modifiers/readonly|readonly]]
	- If the type of the setter parameter is not specified, it is inferred from the return type of the getter

Since TypeScript 4.3, it is possible to have accessors with different types for getting and setting.

```ts
class Thing {
  _size = 0;
 
  get size(): number {
    return this._size;
  }
 
  set size(value: string | number | boolean) {
    let num = Number(value);
 
    // Don't allow NaN, Infinity, etc
 
    if (!Number.isFinite(num)) {
      this._size = 0;
      return;
    }
 
    this._size = num;
  }
}
```


This is a core principle of **Encapsulation**: you hide the raw data and only allow access through controlled "gates."

---

### 2. Why use them instead of public properties?

#### A. Validation

Setters allow you to prevent invalid data from entering your object.

TypeScript

```
set age(value: number) {
  if (value < 0 || value > 150) return; // Ignore invalid age
  this._age = value;
}
```

#### B. Computed Properties (Read-Only)

You can create "virtual" properties that don't actually exist as stored data but are calculated on the fly.

TypeScript

```
class Square {
  constructor(public size: number) {}

  // This property doesn't exist in memory, it's calculated
  get area(): number {
    return this.size * this.size;
  }
}

const sq = new Square(5);
console.log(sq.area); // 25
// sq.area = 100; // ❌ Error: Cannot assign to 'area' because it is a read-only property
```

#### C. Encapsulation / Internal Refactoring

You can change your internal data structure without breaking the code of people using your class. You could change `_firstName` and `_lastName` to a single `_fullName` string internally, but keep the getters the same.

---

### 3. Key TypeScript Rules for Accessors

1. **Implicit `readonly`**: If you have a `get` but no `set`, the property is automatically inferred as `readonly`.
    
2. **Type Consistency**: The return type of the `get` accessor must match the parameter type of the `set` accessor.
    
3. **No Type Annotations on Setters**: You usually don't need to annotate the return type of a setter (it always returns `void` or nothing).
    
4. **Target Version**: To use accessors, your TypeScript configuration (`tsconfig.json`) must target **ES5 or higher**. They are not supported in ES3.
    

---

### 4. How it looks in Memory (Logic Flow)

Accessors act as a middleman between the user of the class and the private data.

---

### 5. Smart Usage: Transforming Data

You can use setters to format data as it comes in.

```ts
class User {
  private _email: string = "";

  get email() {
    return this._email;
  }

  set email(value: string) {
    // Automatically convert email to lowercase before storing
    this._email = value.toLowerCase().trim();
  }
}

const u = new User();
u.email = "  Admin@EXAMPLE.com  ";
console.log(u.email); // "admin@example.com"
```

---

### Summary Comparison

|**Feature**|**Public Property**|**Getter/Setter**|
|---|---|---|
| **Logic**      | None                   | Can run code on read/write        |
| -------------- | ---------------------- | --------------------------------- |
| **Validation** | Not possible           | Full control                      |
| **Read-only**  | Use `readonly` keyword | Provide only `get`                |
| **Storage**    | Occupies memory        | `get` can be computed (no memory) |
