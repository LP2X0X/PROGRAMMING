---
tags: rust, ownership, term, fundamental
---

In Rust, **move** refers to how **ownership** of a value is **transferred** from one variable to another. It’s a core concept of Rust’s memory safety model.

---

## 🧠 What is a “move”?

> In Rust, when a value is **moved**, the original variable can no longer be used — ownership has been **transferred**.

![[trpl04-04.svg|center|350]]

---

## ✅ Example:

```rust
let a = String::from("hello");
let b = a; // 👈 MOVE happens here
```

- `String` is a **heap-allocated** type.
    
- When we assign `a` to `b`, **ownership of the string is moved to `b`**.
    
- After that, **`a` is no longer valid**.
    

```rust
println!("{}", a); // ❌ ERROR: value borrowed after move
```

---

## 🔄 Move vs Copy

|Type|Behavior|Example Types|
|---|---|---|
|**Copy**|Makes a duplicate|`i32`, `bool`, `char`, `f64`|
|**Move**|Transfers ownership|`String`, `Vec`, `Box`, custom structs|

```rust
let x = 5;
let y = x; // ✅ x still usable, because i32 is Copy
```

---

## 📦 When move happens:

1. Assigning a variable: `let b = a;`
    
2. Passing to a function: `do_something(a);`
    
3. Returning from a function: `return a;`
    

---

## 🔐 Why does Rust do this?

To **ensure memory safety** without garbage collection.  
If multiple variables had ownership, one might free it while others still use it — causing crashes.

---

## 🔧 What if I want to keep using the original?

Use `.clone()` to make a deep copy:

```rust
let a = String::from("hi");
let b = a.clone(); // ✅ a and b are both valid
```

---

## 🧠 Summary

> A **move** means **ownership is transferred**.  
> After the move, the original variable is **no longer usable**.

It’s how Rust enforces safety without a garbage collector.
