---
tags: rust, datatype, struct, fundamental
---

In Rust, a `struct` (short for **structure**) is a custom data type that **groups related values** together into a single type — like classes or records in other languages.

---

## 🧠 Why use a `struct`?

- To group multiple pieces of data into one unit
    
- To give data **meaningful names**
    
- To model real-world things in your code (e.g., `User`, `Point`, `Rectangle`)
    

---

## ✅ Basic Syntax

- To define a struct, we enter the keyword `struct` and name the entire struct. A struct’s name should describe the significance of the pieces of data being grouped together. Then, inside curly brackets, we define the names and types of the pieces of data, which we call _fields_.

```rust
struct User {
    username: String, // field
    age: u32,
    active: bool,
}
```

---

## 🛠 Creating an Instance

```rust
let user1 = User {
    username: String::from("alice"),
    age: 30,
    active: true,
};
```

You can access fields like this:

```rust
println!("Username: {}", user1.username);
```

---

## ✨ Mutability

To modify fields, the entire instance must be `mut`:

```rust
let mut user2 = User {
    username: String::from("bob"),
    age: 25,
    active: false,
};

user2.age += 1;
```

---

## 🧱 Struct Update Syntax

Reuse values from another struct:

```rust
let user3 = User {
    username: String::from("charlie"),
    ..user1  // copies age and active from user1
};
```

---

## 🧩 Tuple Structs

Like tuples, but with named types:

```rust
struct Color(u8, u8, u8);

let red = Color(255, 0, 0);
println!("{}", red.0); // 255
```

---

## 📦 Unit-Like Structs

Useful for marker traits:

```rust
struct Marker;
```

---

## 🔧 With `impl` Block (methods)

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
```

```rust
let rect = Rectangle { width: 10, height: 5 };
println!("Area: {}", rect.area());
```

---

## 🧠 Summary

|Concept|Syntax|
|---|---|
|Define struct|`struct Name { field: Type, ... }`|
|Create value|`let x = Struct { field: value }`|
|Access field|`x.field`|
|Add methods|`impl Struct { fn ... }`|

---

Let me know if you'd like help with:

- Lifetimes in struct fields
    
- Structs with references
    
- Using `derive` macros (like `Debug`, `Clone`)