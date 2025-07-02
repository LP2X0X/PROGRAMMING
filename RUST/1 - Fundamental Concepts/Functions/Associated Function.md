---
tags: rust, function, fundamental
---

In Rust, an **associated function** is a function defined in an `impl` block **without** taking `self`, `&self`, or `&mut self` as its first parameter.

---

## 🧩 What is an Associated Function?

* It belongs to a type (struct, enum, or trait), but **is not called on an instance**.
* It's called using the type name, like `Type::function()`.
* Often used for **constructors**, utility functions, or constants.

---

### ✅ Basic Example

```rust
struct Circle {
    radius: f64,
}

impl Circle {
    // Associated function — no `self`
    fn new(radius: f64) -> Self {
        Self { radius }
    }

    // Method — has `&self`
    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }
}

fn main() {
    let c = Circle::new(2.5);       // Associated function
    println!("Area: {}", c.area()); // Method
}
```

```ad-note
The `new` keyword isn’t a special name and isn’t built into the language.
```

```ad-note
The `Self` keywords in the return type and in the body of the function are aliases for the type that appears after the `impl` keyword, which in this case is `Circle`.
```

---

## 🔍 Associated Function vs Method

| Feature             | Associated Function   | Method                        |
| ------------------- | --------------------- | ----------------------------- |
| Has `self`?         | ❌ No                  | ✅ Yes (`self`, `&self`, etc.) |
| Called on instance? | ❌ Called on type      | ✅ Called on value             |
| Use case            | Constructors, helpers | Actions on the instance       |

---

### 🧠 Why are they called "associated"?

Because they are:

* **Not free-standing functions**
* But **tied to a type** through `impl`

---

## 🛠️ Associated Functions in Traits

Associated functions can also be defined in traits:

```rust
trait Shape {
    fn new() -> Self; // associated function in a trait
}

struct Square {
    side: f64,
}

impl Shape for Square {
    fn new() -> Self {
        Self { side: 1.0 }
    }
}
```

---

## 🎯 Associated Constants (Bonus)

Rust also supports **associated constants** inside `impl`:

```rust
impl Circle {
    const PI: f64 = 3.14159;
}
```

Call it with:

```rust
println!("π ≈ {}", Circle::PI);
```

---

## ✅ Summary

| Feature           | Associated Function                |
| ----------------- | ---------------------------------- |
| Defined in `impl` | Yes                                |
| Requires `self`?  | No                                 |
| Called like       | `Type::function()`                 |
| Common use cases  | Constructors, constants, utilities |