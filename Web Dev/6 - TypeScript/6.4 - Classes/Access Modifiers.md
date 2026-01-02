---
tags: 
 - typescript
 - class
 - access
 - property
---

## `public` Properties

### What `public` Means

`public` properties are **accessible from anywhere**:

- Inside the class
    
- From subclasses
    
- From outside the class
    

```ts
class User {
  public name: string;

  constructor(name: string) {
    this.name = name;
  }
}

const u = new User("Long");
u.name;        // ✅ allowed
u.name = "Pham"; // ✅ allowed
```

### Key Notes

- `public` is the **default** visibility
    
- You can omit the keyword
    

```ts
class User {
  name: string; // public by default
}
```

---

## `private` Properties

### What `private` Means

`private` properties are **only accessible within the declaring class**.

```ts
class User {
  private password: string;

  constructor(password: string) {
    this.password = password;
  }

  check(pw: string) {
    return this.password === pw; // ✅ allowed
  }
}

const u = new User("1234");
u.password; // ❌ Error
```

### Important: TypeScript `private` Is Compile-Time

```ts
class A {
  private x = 1;
}

const a = new A();
// a["x"]; // ❌ TypeScript error, but exists at runtime
```

At runtime, `x` still exists as a normal property.

---

## `private` vs `#private` (JavaScript)

```ts
class A {
  #x = 1;
}
```

| Feature                | `private`    | `#private` |
| ---------------------- | ------------ | ---------- |
| Enforced at            | Compile time | Runtime    |
| Accessible via bracket | Yes          | No         |
| JavaScript native      | No           | Yes        |

Use `#private` when **runtime privacy is required**.

---

## Access Modifiers in Constructor Parameters

TypeScript allows declaring properties directly in the constructor.

```ts
class User {
  constructor(
    public name: string,
    private password: string
  ) {}
}
```

This both:

- Declares the properties
    
- Assigns them
    

---

## Interaction with Inheritance

### `public`

```ts
class Base {
  public id = 1;
}

class Child extends Base {
  print() {
    console.log(this.id); // ✅ allowed
  }
}
```

### `private`

```ts
class Base {
  private secret = "x";
}

class Child extends Base {
  test() {
    // this.secret; ❌ Error
  }
}
```

`private` members are **not accessible in subclasses**.

---

## Common Design Pattern

Use `private` + accessor methods:

```ts
class Account {
  private balance = 0;

  getBalance() {
    return this.balance;
  }
}
```

Encapsulation is enforced by the type system.

---

## Summary

- `public` = accessible everywhere (default)
    
- `private` = accessible only inside the class
    
- TypeScript `private` is compile-time only
    
- Use `#private` for true runtime privacy
    
- Constructor parameter properties reduce boilerplate
    
