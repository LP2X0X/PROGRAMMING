---
tags: rust, datatype, enum, fundamental
---

## 🧱 What is `Option<T>`?

- `Option` is an [[RUST/1 - Fundamental Concepts/Data Types/Enums/Enum|enum]] defined by the standard library.

```rust
enum Option<T> {
    Some(T),
    None,
}
```

- `Some(T)`: holds a value of type `T`
- `None`: represents no value

```ad-note
The `Option<T>` enum is so useful that it’s even included in [[The Prelude|the prelude]]; you don’t need to bring it into scope explicitly.
```

---

## 🚫 Why Rust Uses `Option` (No `null`!)

Most languages use `null` or `nil`, which leads to runtime errors (null dereference, crashes, etc.).

Rust solves this by **removing null entirely** and using `Option<T>` to **explicitly represent absence** at the type level — enforced by the compiler.

---

## ✅ Common Use Cases

- **Optional input or configuration**
    
- **Searching for something that might not exist**
    
- **Failure without an error (e.g., index out of range)**
    
- **Possibly empty values (e.g., environment variables)**
    

---

## 📌 Basic Example

```rust
let name: Option<String> = Some("Alice".to_string());
let age: Option<u32> = None;
```

---

## 🧠 Pattern Matching

```rust
match name {
    Some(n) => println!("Name: {}", n),
    None => println!("No name provided."),
}
```

Or with `if let` (more concise):

```rust
if let Some(n) = name {
    println!("Hi, {}", n);
}
```

---

## 🛠 Useful Methods

### 🔍 Inspect

```rust
option.is_some();     // true if it's Some
option.is_none();     // true if it's None
```

### 🧱 Get the value

```rust
option.unwrap();      // gets the value, panics if None
option.unwrap_or("guest");  // fallback value
```

### 🔄 Transform

```rust
let opt = Some("hello");
let len = opt.map(|s| s.len());  // Some(5)
```

### 🔄 Chain fallbacks

```rust
option.or(Some("default".to_string()));
```

### 🔄 Convert  from reference to value

```rust
let a: Option<&i32> = Some(&10);
let b: Option<i32> = a.copied(); // now it's Some(10)
```

---

## 🔁 Converting Between Types

```rust
fn get_username(id: u32) -> Option<String> {
    if id == 1 {
        Some("Alice".to_string())
    } else {
        None
    }
}

let user = get_username(2).unwrap_or("guest".to_string());
```

---

## ✅ When to Use `Option<T>`

Use it when **not having a value is expected and okay**, like:

- Optional function return
    
- Config values
    
- Missing data
    
- Optional struct fields
    

Use `Result<T, E>` when failure **might need an error message** or recovery logic.

---

## 🔐 Guarantees

Rust forces you to **handle `None`**, so:

- No null pointer dereferencing
    
- No undefined behavior from missing values
    
- No silent bugs — compiler helps you
    

---

## 🧠 Summary

|Concept|`Option<T>`|
|---|---|
|Meaning|Value might be missing|
|Variants|`Some(value)`, `None`|
|Replaces|Nulls in other languages|
|Enforced by|Rust compiler|
|Safer than|Nulls, unchecked optionals|
