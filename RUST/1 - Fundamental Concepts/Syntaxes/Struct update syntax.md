---
tags:
  - rust
  - syntax
---

In Rust, **struct update syntax** lets you create a new instance of a struct by specifying new values for some fields and copying the rest from an existing instance.

#### ✅ Basic Example:

```rust
struct User {
    username: String,
    email: String,
    active: bool,
}

fn main() {
    let user1 = User {
        username: String::from("alice"),
        email: String::from("alice@example.com"),
        active: true,
    };

    let user2 = User {
        email: String::from("bob@example.com"),
        ..user1
    };

    println!("{}", user2.username); // "alice"
    println!("{}", user1.email); // Still valid since its value was not moved out
}
```

#### 🔍 Explanation:

- `..user1` copies all the remaining fields from `user1` that weren't specified explicitly (`username` and `active`).
    
- `email` is overridden in `user2`.
    
- After this, the `username` field of `user1` is **moved**, so it can no longer be used. However, the `email` field is explicitly overridden in `user2`, so it remains untouched in `user1`. Since the `active` field implements the `Copy` trait, it will be copied rather than moved, meaning it can still be accessed from `user1`.
    

---

### 🛑 Important Notes:

- This only works if the struct is **not tuple-like or unit-like**.
    
- You can't use `..other_struct` inside a tuple struct or enum.
    
- If the struct has non-`Copy` fields like `String`, ownership will be moved unless the field type implements `Clone`.
    