---
tags:
  - rust
  - term
  - fundamental
---

In Rust, **shadowing** is a technique that lets you **declare a new variable with the same name** as a previous one, effectively **replacing it in the current scope**.

This is **not mutation**, but **re-binding**. It’s a powerful feature and is used often for clarity, transformation, and type changes.

---

## 🔹 Basic Example of Shadowing

```rust
fn main() {
    let x = 5;
    let x = x + 1; // shadowed
    let x = x * 2; // shadowed again

    println!("x = {}", x); // x = 12
}
```

- First `x = 5`
    
- Then shadowed with `x + 1 = 6`
    
- Then shadowed with `x * 2 = 12`
    

✅ Each `let x = ...` creates a **new immutable binding** named `x`.

---

## 🔹 Shadowing vs Mutation

```rust
fn main() {
    let mut y = 5;
    y = y + 1; // mutable update

    let y = y * 2; // shadowing!
    println!("y = {}", y); // y = 12
}
```

|Operation|Keyword|Behavior|
|---|---|---|
|`let mut y =`|mutable|change in-place|
|`let y =`|shadow|new binding|

> Shadowing lets you **change types**, but mutation does **not**.

---

## 🔹 Shadowing to Change Type

```rust
fn main() {
    let s = "42";
    let s = s.parse::<i32>().unwrap(); // s is now an i32

    println!("s + 1 = {}", s + 1); // s = 43
}
```

- First `s` is a `&str`
    
- Then it becomes an `i32` via parsing and shadowing
    

✅ Shadowing is **the idiomatic way** to transform data and change type in Rust.

---

## 🔹 Shadowing Inside Scopes

```rust
fn main() {
    let a = 10;

    {
        let a = 20;
        println!("Inner a = {}", a); // 20
    }

    println!("Outer a = {}", a); // 10
}
```

- Shadowing is **scope-local**.
    
- Inner shadowing does **not affect the outer binding**.
    

---

## ✅ When to Use Shadowing

|Use case|Benefit|
|---|---|
|Transform value (clean style)|No need to invent new names|
|Change type|Mutation can’t do this|
|Reset or override for scope|Keeps things clean|

---

Let me know if you'd like real-world examples of shadowing in:

- CLI argument parsing
    
- File I/O
    
- Pattern matching
    
- Loops or error handling