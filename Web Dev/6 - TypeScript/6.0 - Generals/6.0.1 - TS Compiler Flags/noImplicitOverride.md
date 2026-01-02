---
tags: 
 - typescript
 - compiler
 - flag
---

The `noImplicitOverride` flag is a compiler setting in TypeScript (found in your `tsconfig.json`) designed to make class inheritance safer and more predictable.

When this flag is enabled, it forces you to be explicit about which methods in a child class are intended to **override** methods from the parent class.

---

### The Problem: Accidental Overrides

Without this flag, if you create a method in a child class with the same name as a method in the parent, you override it automatically. This can lead to two main issues:

1. **Accidental Shadowing:** You name a method `render()` not realizing the parent already has a `render()` method that does something critical.
    
2. **Fragile Base Classes:** If a library update adds a new method to a parent class that happens to have the same name as your child class method, your code’s behavior might suddenly change without warning.
    

---

### How it Works

When `noImplicitOverride` is set to `true`, you **must** use the `override` keyword when you want to replace a parent's method.

#### ❌ Without the `override` keyword (Error)

```ts
class Shape {
  draw() {
    console.log("Drawing a shape");
  }
}

class Circle extends Shape {
  // ❌ Error: This member must have an 'override' modifier because 
  // it overrides a member in the base class 'Shape'.
  draw() {
    console.log("Drawing a circle");
  }
}
```

#### ✅ With the `override` keyword (Correct)

```ts
class Circle extends Shape {
  override draw() {
    console.log("Drawing a circle");
  }
}
```

---

### The "Safety Check" Benefit

The real power of `override` isn't just forcing you to type more; it’s a **contract**. If you mark a method as `override`, but that method _doesn't actually exist_ in the parent class, TypeScript will throw an error.

This is extremely helpful during **refactoring**:

```ts
class Shape {
  // Imagine you rename 'draw' to 'paint'
  paint() { ... }
}

class Circle extends Shape {
  // ❌ Error: This member cannot have an 'override' modifier 
  // because it is not declared in the base class 'Shape'.
  override draw() { 
    console.log("Drawing a circle");
  }
}
```

Because of the error, you immediately realize that `Circle` is no longer correctly overriding the parent logic.

---

### Comparison Table

| **Feature**        | **Standard TypeScript**             | **With noImplicitOverride: true**                 |
| ------------------ | ----------------------------------- | ------------------------------------------------- |
| **New methods**    | Just write the method.              | Just write the method.                            |
| **Overriding**     | Happens automatically by name.      | Requires the `override` keyword.                  |
| **Parent Changes** | Can break child logic silently.     | Throws error if parent method is deleted/renamed. |
| **Readability**    | Hard to tell which methods are new. | Clearly identifies "extended" vs "new" logic.     |

---

### Why you should turn it on

It is highly recommended for professional projects because:

1. It serves as **documentation**: Anyone reading the child class knows exactly which methods are part of the inherited API.
    
2. It provides **refactor safety**: It prevents "ghost methods" that you think are overrides but aren't (due to typos or parent class changes).
    
