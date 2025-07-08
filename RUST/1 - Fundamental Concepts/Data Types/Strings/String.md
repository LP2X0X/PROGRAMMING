---
tags: rust, datatype, fundamental
---


### ✅ 1. Definition

- The **String** type manages data allocated on the heap and as such is able to store an amount of text unknown at compile time.
- Strings in Rust are UTF-8 encoded.

```ad-note
`String` is actually implemented as a wrapper around a vector of bytes with some extra guarantees, restrictions, and capabilities.
```

---

### 🛠 2. **Creating a `String`**

- You can create a `String` from a string literal using the `from` function, like so:

```rust
let s = String::from("hello");
let s = "hello".to_string();
```

---

### ➕ 3. **Updating a `String`**

- Append with `.push_str("...")` or `.push('c')`
    
```rust
let mut s = String::from("hello");

s.push_str(", world!"); // push_str() appends a literal to a String // it use string slice
s.push('l'); // The `push` method takes a single character as a parameter and adds it to the `String`.

println!("{s}"); // This will print `hello, world!`
```

- Concatenate with `+` or `format!`:
    
```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // s1 is moved
```

```ad-warning
Clearing a string in Rust **is a mutable action** — because it **modifies the content** of the string.
```

---

### 🧠 4. **Strings Are UTF-8**

Rust strings are UTF-8 encoded, so characters may take multiple bytes.

```rust
let hello = "Здравствуйте"; // 24 bytes, 12 characters
```

---

### ❌ 5. **Indexing Is Not Allowed**

You **cannot** access a character directly with indexing (`s[0]`) because characters may be multiple bytes. Instead, use:

- `.chars()` → iterate over characters
    
- `.bytes()` → iterate over raw bytes
    

---

### 🔍 6. **Slicing Strings**

Slicing must happen at valid **character boundaries** (not arbitrary bytes), or it will panic.

```rust
let s = "Здравствуйте";
let slice = &s[0..4]; // OK (4 bytes = 2 characters)
```

---

### 📋 7. **Iterating Over Strings**

```rust
for c in "hello".chars() {
    println!("{}", c);
}
```

Use `.chars()` for Unicode characters, `.bytes()` for raw bytes.

---

### 🚫 8. **String vs Other Languages**

Rust’s String model prioritizes safety and correctness over convenience like in Python or JavaScript.
