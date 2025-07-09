---
tags: rust, distinguish
---

In Rust, `chars()` and `bytes()` are methods on `&str` (string slices), and they represent **two different ways to iterate** over the contents of a string:

---

## 🔤 `chars()` – Unicode Scalar Values (Human-readable)

- Iterates over **Unicode scalar values** (i.e., characters you can see).
    
- Each item is a `char` (4 bytes, Unicode scalar).
    

```rust
let s = "hêllo";
for c in s.chars() {
    println!("{}", c);
}
```

**Output:**

```
h
ê
l
l
o
```

- `ê` is one character (char), even though it takes multiple bytes.
    

---

## 🧱 `bytes()` – Raw UTF-8 Bytes (Machine-level)

- Iterates over the **underlying bytes** (`u8`) of the UTF-8 encoded string.
    
- Each item is a `u8` (1 byte).
    

```rust
let s = "hêllo";
for b in s.bytes() {
    println!("{}", b);
}
```

**Output (example):**

```
104 195 170 108 108 111
```

> - `ê` is encoded as two bytes: `195` and `170`.
>     

---

## 📌 Summary Table

|Method|Yields|Type|Use case|
|---|---|---|---|
|`chars()`|Characters|`char`|When working with visible text or graphemes|
|`bytes()`|Raw bytes|`u8`|When working with encoding, networking, or hashing|

---

## ⚠️ Warning:

Indexing a string like `s[1]` is not allowed in Rust, because strings are UTF-8 and indexing could split a character. Use `.chars().nth(i)` or slicing with care.
