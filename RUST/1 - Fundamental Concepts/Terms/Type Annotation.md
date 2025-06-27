---
tags: rust, term, fundamental
---

In Rust, **type annotations** are how you explicitly specify the type of a variable, function parameter, or return value. Although Rust has powerful **type inference**, sometimes you must (or should) annotate the type — especially when the compiler can’t guess it or to improve code clarity.

---

## ✅ Type Annotation Syntax

### 🟢 Variables

```rust
let x: i32 = 42;
let name: &str = "Rust";
let pi: f64 = 3.14;
```

### 🟢 Functions

```rust
fn square(n: i32) -> i32 {
    n * n
}
```

### 🟢 Function with Multiple Types

```rust
fn greet(name: &str, age: u32) -> String {
    format!("Hello, {name}, age {age}")
}
```

---

## 🧠 Why Use Type Annotations?

|Purpose|Example|
|---|---|
|🛠 Required when type can't be inferred|`let guess: u32 = "42".parse().unwrap();`|
|📘 Improves readability|`let status: Result<(), Error>`|
|🔬 Guides generic functions|`let nums = Vec::<i32>::new();`|

---

## 🧪 Examples

### Type annotation in `parse`:

```rust
let n: i32 = "5".parse().unwrap();
```

If you omit the type here, Rust won’t know what type to parse into.

### With generic collections:

```rust
let names: Vec<String> = vec!["Alice".to_string(), "Bob".to_string()];
```

---

## 🟡 Type Inference vs Annotation

Rust often infers the type:

```rust
let x = 42;  // inferred as i32
```

But you **must annotate** when:

- Using `parse()`, `collect()`, etc.
    
- Passing to functions with generic params
    
- Working with trait objects or boxed values
    

---

## 🧠 Tip: Annotate Only When Needed

Over-annotating can make code noisy. Let Rust infer types when it’s obvious, and annotate when:

- The compiler complains
    
- You want to document intent
    
- You’re using traits, generics, or advanced features
    
