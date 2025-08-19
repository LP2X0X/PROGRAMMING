---
tags:
  - rust
  - control
  - flow
  - fundamental
---

In Rust, the `for` loop is a powerful and safe way to **iterate over values** — especially ranges, collections like vectors, or iterators.

---

## 🔁 Basic Syntax

```rust
for item in iterable {
    // do something with item
}
```

Rust’s `for` loop automatically uses the `.into_iter()` trait behind the scenes.

---

## ✅ Common `for` loop use cases:

### 🔢 1. Loop over a range

```rust
for i in 0..5 {
    println!("i = {}", i);
}
```

- `0..5` is a **range** that includes 0 up to (but not including) 5.
    
- If you want to **include 5**, use `0..=5`.
    

---

### 📦 2. Loop over a vector

```rust
let names = vec!["Alice", "Bob", "Carol"];

for name in names {
    println!("Hello, {}", name);
}
```

⚠️ Note: This **moves** the values out of `names`.  
If you want to keep the vector usable afterward, **borrow** it:

```rust
for name in &names {
    println!("Hello, {}", name);
}
```

Use `&mut names` to mutate items during iteration.

---

### 🧮 3. Loop with index using `.enumerate()`

```rust
let nums = vec![10, 20, 30];

for (i, value) in nums.iter().enumerate() {
    println!("Index {} has value {}", i, value);
}
```

---

### 🔄 4. Nested loops

```rust
for i in 0..3 {
    for j in 0..2 {
        println!("({}, {})", i, j);
    }
}
```

---

## 🔚 `break` and `continue`

- `break` exits the loop early
    
- `continue` skips to the next iteration
    

```rust
for i in 0..5 {
    if i == 3 {
        break;
    }
    if i == 1 {
        continue;
    }
    println!("{}", i);
}
```

---

## 🎯 Loop labels (for breaking nested loops)

```rust
'outer: for x in 0..3 {
    for y in 0..3 {
        if x + y == 3 {
            break 'outer;
        }
    }
}
```

---

## 🧠 Summary

|You want to...|Use this|
|---|---|
|Loop over a range|`for i in 0..10`|
|Loop over a collection|`for item in &vec` or `vec.iter()`|
|Get index and value|`.enumerate()`|
|Mutate values|`for val in &mut vec`|
|Break or skip iteration|`break`, `continue`|
|Exit outer loop from inner|Use `'label: for` and `break 'label`|
