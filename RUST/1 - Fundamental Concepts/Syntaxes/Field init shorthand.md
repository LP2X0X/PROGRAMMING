---
tags:
  - rust
  - syntax
---

In Rust, **field init shorthand** lets you **avoid repeating field names** when initializing a struct — if your variable names match the field names.

---

## ✅ Syntax

```rust
struct User {
    username: String,
    age: u32,
}

fn build_user(username: String, age: u32) -> User {
    User {
        username, // shorthand for username: username
        age,      // shorthand for age: age
    }
}
```

---

## 🧠 Without shorthand

You’d normally have to write:

```rust
User {
    username: username,
    age: age,
}
```

But with field init shorthand, Rust will **fill in the rest** if the variable name matches the field name.

---

## 🧾 When is it allowed?

Only when:

- The variable name is the **same** as the field name.
    
- You're inside a scope (like a function) where those variable names exist.
    

---

## 🧪 Example

```rust
let username = String::from("alice");
let age = 30;

let user = User { username, age }; // shorthand works here
```

---

## 🚫 Invalid Case

```rust
let name = String::from("alice");
User { username, age } // ❌ won't work — there's no variable `username`
```

You must have an **exact name match**.

---

## ✅ Summary

|Without shorthand|With shorthand|
|---|---|
|`field: variable`|`field`|
|`username: username`|`username`|
|`age: age`|`age`|

Use it for cleaner, less repetitive code when your variable names match struct field names.