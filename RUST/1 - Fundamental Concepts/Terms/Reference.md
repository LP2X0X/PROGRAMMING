---
tags: rust, term, fundamental
---

In Rust, a **reference** lets you **borrow** a value instead of taking ownership of it. It’s one of the most important features in Rust’s safety system.

---

## 🧠 What Is a Reference?

A reference is like a pointer to a value — it lets you **read or write** to a variable **without moving it**.

```rust
let x = 10;
let r = &x;  // r is a reference to x

println!("Value of x is: {}", r);
```

- `&x` means “a reference to `x`”
- `x` is not moved — you can still use it
- `r` just **borrows** `x` temporarily

```ad-note
You can **read** through a reference as is, but to **write** (i.e., mutate), you must **dereference** it.
```

---

## ✅ Types of References

|Type|Syntax|Meaning|
|---|---|---|
|Shared|`&T`|Read-only reference|
|Mutable|`&mut T`|Allows modifying the value|

---

### 🔹 Immutable Reference (`&T`)

```rust
let name = String::from("Rust");
let ref1 = &name;
let ref2 = &name;

println!("{ref1} and {ref2}"); // ✅ You can have many immutable refs
```

---

### 🔸 Mutable Reference (`&mut T`)

````ad-question
Why do you need mutable reference, can't you just modify it directly?
```ad-Answer
You **can** — but only **inside the same scope** where you own the value. The moment you need to modify it **from another function or context**, you **must use a mutable reference**.
```
````

```rust
let mut value = 5;
let ref_mut = &mut value;
*ref_mut += 1;

println!("{value}"); // 6
```

❗ You can only have **one mutable reference** at a time.

- The benefit of having this restriction is that Rust can prevent data races at compile time. A _data race_ is similar to a race condition and happens when these three behaviors occur:
	- Two or more pointers access the same data at the same time.
	- At least one of the pointers is being used to write to the data.
	- There’s no mechanism being used to synchronize access to the data.

---

## 🧱 Rules of Borrowing (VERY IMPORTANT)

1. ✅ You can have **many `&T` (immutable)** OR
    
2. ✅ You can have **one `&mut T` (mutable)**
    
3. ❌ You **cannot mix** mutable and immutable references at the same time.
    

```rust
let mut data = String::from("hello");
let r1 = &data;
let r2 = &data;
// let r3 = &mut data; // ❌ ERROR: cannot borrow as mutable because it's already borrowed as immutable
```

---

## 🎯 Why use references?

- **Avoid copying/moving** large data
    
- **Access values temporarily**
    
- **Share values safely without ownership issues**
    
- **Enable efficient and safe code**
    

---

## 🔬 Scope

- **Note that a reference’s scope starts from where it is introduced and continues through the last time that reference is used**. 
- For instance, this code will compile because the last usage of the immutable references is in the `println!`, before the mutable reference is introduced:

```rust
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!("{r1} and {r2}");
    // Variables r1 and r2 will not be used after this point.

    let r3 = &mut s; // no problem
    println!("{r3}");
```

---

## 🧪 Function Example:

```rust
fn print_message(msg: &String) {
    println!("Message: {msg}");
}

fn main() {
    let greeting = String::from("Hi");
    print_message(&greeting); // ✅ pass by reference, ownership kept
    println!("{greeting}");   // ✅ still usable
}
```

---

## 📦 Summary

|Concept|Example|
|---|---|
|Create ref|`let r = &x;`|
|Dereference|`*r`|
|Mutable ref|`let r = &mut x;`|
|Function param|`fn foo(x: &i32)`|
