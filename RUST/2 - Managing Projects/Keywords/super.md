---
tags:
  - rust
  - keyword
  - fundamental
---

In Rust, the `super` keyword refers to the **parent module** — it lets you access items from one level up in the module hierarchy.

---

## 🧱 Think of it like file paths

Just like:

- `./` = current directory
    
- `../` = parent directory
    

In Rust modules:

- `self` = current module
    
- `super` = parent module
    
- `crate` = root of the crate
    

---

## ✅ Example

### File structure:

```bash
src/
├── lib.rs
└── network/
    ├── mod.rs
    └── protocol.rs
```

### `lib.rs`

```rust
pub mod network;
fn log(msg: &str) {
    println!("LOG: {msg}");
}
```

### `network/mod.rs`

```rust
pub mod protocol;
```

### `network/protocol.rs`

```rust
fn handle_protocol() {
    super::super::log("Handling protocol");
}
```

- `super` → goes up to `network`
    
- `super::super` → goes up to crate root (`lib.rs`)
    

✅ Now the private `log()` function in `lib.rs` is accessible!

---

## 🔹 Common uses of `super`

|Use case|Example|
|---|---|
|Access item in parent module|`super::some_fn()`|
|Access sibling module|`super::sibling_mod::some_fn()`|
|Re-export parent function|`pub use super::helper;`|

---

## ⚠️ Note

- You can **chain `super`**: `super::super::...`
    
- Works the same in inline modules and in files
    

---

## 🧠 Summary

|Keyword|Refers to|Used for|
|---|---|---|
|`self`|current module|calling functions within the module|
|`super`|parent module|calling or referencing upper-level items|
|`crate`|root of the crate|referencing global crate items|
