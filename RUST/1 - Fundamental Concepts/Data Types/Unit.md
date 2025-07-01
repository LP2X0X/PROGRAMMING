---
tags: rust, datatype, fundamental
---

- In Rust, **`unit`** refers to the **unit type**, written as `()` — a type that represents "nothing" or "no meaningful value".

---

### 🧱 What is the unit type `()`?

- It is a **type** with exactly **one value**, also written as `()`.
    
- Similar to `void` in C/C++ or `None` in Python (but with an actual value).
    
- It is used when a function or expression returns nothing meaningful.
    

---

### ✅ Examples

#### 1. **Functions that return nothing:**

```rust
fn do_something() {
    println!("This returns nothing!");
    // Implicitly returns ()
}
```

The return type here is `()` by default.

You could write it explicitly:

```rust
fn do_something() -> () {
    println!("Still returns unit");
}
```

---

#### 2. **The unit value in practice:**

```rust
let x: () = ();  // Valid: x is of unit type, and its value is ()
```

---

#### 3. **Used in closures or match arms that don't return a value:**

```rust
let result = match 5 {
    1 => "one",
    _ => {
        println!("not one");
        ()
    }
};
```

---

### 🧠 Why is it useful?

- Acts as a "do nothing" or "no value" marker.
    
- Needed because Rust is expression-based, so even "no result" needs a type.
    
- Makes functions with side effects type-safe (they return `()` rather than nothing).
    
