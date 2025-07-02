---
tags: rust, function, fundamental
---

In Rust, a **method** is a function **associated with a struct, enum, or trait**, and is called using **dot notation** (like `obj.method()`).

---

## 🧱 Basic Method Syntax

You define methods inside an `impl` block (implementation block):

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Method that takes &self
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
```

### ✅ Calling the method:

```rust
let rect = Rectangle { width: 10, height: 20 };
println!("Area: {}", rect.area()); // Calls rect.area()
```

```ad-tip
Note that we can choose to give a method the same name as one of the struct’s fields. This is handy when we want the method to only return the value in the field and do nothing else.
```

---

## 📌 Method Types in Rust

### 1. **Instance Methods**

#### a. Takes `&self` (shared reference)

Does not change the instance:

```rust
fn describe(&self) { ... }
```

```ad-note
The `&self` is actually short for `self: &Self`.
```

#### b. Takes `&mut self` (mutable reference)

Can modify the instance:

```rust
fn grow(&mut self, value: u32) {
    self.width += value;
}
```

#### c. Takes `self` (takes ownership)

Consumes the instance:

```rust
fn destroy(self) {
    println!("Destroyed: {}x{}", self.width, self.height);
}
```

---

### 2. **Associated Functions (like static methods)**

They don’t take `self` — called on the type itself:

```rust
impl Rectangle {
    fn new(width: u32, height: u32) -> Self {
        Self { width, height }
    }
}

let r = Rectangle::new(30, 40);
```

---

## 🧠 Traits and Methods

If you define methods in a `trait`, they can be overridden or implemented by types:

```rust
trait Speak {
    fn speak(&self);
}

struct Dog;

impl Speak for Dog {
    fn speak(&self) {
        println!("Woof!");
    }
}
```

---

## 🧪 Full Example

```rust
struct Counter {
    value: i32,
}

impl Counter {
    fn new() -> Self {
        Self { value: 0 }
    }

    fn increment(&mut self) {
        self.value += 1;
    }

    fn get(&self) -> i32 {
        self.value
    }
}

fn main() {
    let mut c = Counter::new();
    c.increment();
    println!("Value: {}", c.get());
}
```

---

## 📌 Summary

| Signature     | Meaning                           |
| ------------- | --------------------------------- |
| `&self`       | Read-only access                  |
| `&mut self`   | Mutable access                    |
| `self`        | Takes ownership (consumes `self`) |
| `fn new(...)` | Associated (static) function      |