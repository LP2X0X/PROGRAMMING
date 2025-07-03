---
tags: rust, keyword, fundamental
---

The `pub` keyword in Rust is used to make **items accessible outside their current module** — it controls **visibility**.

---

## 🧱 By default: everything is private in Rust

If you don't use `pub`, your functions, structs, modules, etc., are **only accessible within the current module**.

---

## ✅ What `pub` does

|Use case|Meaning|
|---|---|
|`pub fn`|Function is accessible from outside the module|
|`pub struct`|Struct is accessible from outside|
|`pub field_name: Type`|Struct field is accessible (needs `pub` **per field**)|
|`pub mod`|Makes a submodule accessible|
|`pub use`|Re-exports an item so others can use it from this module|

---

### 🔹 Examples

```rust
mod hidden {
    pub fn hello() {
        println!("Hello!");
    }
}

fn main() {
    // hidden::hello(); ❌ Error! hidden is private
}
```

But with `pub mod`:

```rust
pub mod hidden {
    pub fn hello() {
        println!("Hello!");
    }
}

fn main() {
    hidden::hello(); // ✅ Works now
}
```

---

### 🔸 Struct field visibility

```rust
pub struct Point {
    pub x: i32,
    y: i32, // private!
}

fn main() {
    let p = Point { x: 1, y: 2 }; // ❌ Can't access y outside its module
}
```

You must mark **each field** `pub` if you want it public.

---

## 🧠 Other visibility options

Rust has more fine-grained visibility control:

|Visibility keyword|Meaning|
|---|---|
|`pub`|Public everywhere|
|`pub(crate)`|Visible only within the current crate|
|`pub(super)`|Visible in the parent module|
|`pub(in path)`|Visible only in a specific module path|

---

### 🔹 Example with `pub(crate)`

```rust
pub(crate) fn internal_api() {
    // usable inside the crate, not from external crates
}
```

---

## 🧠 Summary

|Without `pub`|Only accessible within the current module|
|---|---|
|With `pub`|Accessible from outside (like public API)|
|With `pub(crate)`|Accessible throughout the crate, but not public|
|With `pub(super)`|Accessible to the parent module|
