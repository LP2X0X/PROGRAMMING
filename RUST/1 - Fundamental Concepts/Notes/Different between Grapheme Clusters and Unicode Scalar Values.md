---
tags: rust, distinguish
---

## 🧱 1. Unicode Scalar Values

### ✅ What they are:

- A **Unicode scalar value** is any valid Unicode **code point**, **except** for surrogate code points (U+D800 to U+DFFF).
    
- Think of them as the **basic unit of stored text**.
    

### 🔠 Example:

The letter **`é`** (U+00E9) is a single Unicode scalar value.

But you could also represent it as:

- `e` (U+0065) + `́` (U+0301) → this is **two scalar values** (base + combining accent).
    

---

## 🧩 2. Grapheme Clusters

### ✅ What they are:

- A **grapheme cluster** is what **users perceive as a single character**.
    
- It can be **one or more Unicode scalar values** that combine into a visible character.
    
- Also called **"user-perceived characters"**.
    

### 🔠 Example:

`é` = `e` (U+0065) + `́` (U+0301)

- 2 **scalar values**
    
- 1 **grapheme cluster**
    

So:

```rust
"é".chars().count();      // => 2 (scalar values)
"é".graphemes(true).count(); // => 1 (grapheme cluster, using unicode-segmentation crate)
```

---

## 📌 In Rust

|Concept|API / Trait|Notes|
|---|---|---|
|**Unicode scalar**|`.chars()`|Built into Rust stdlib|
|**Grapheme cluster**|`.graphemes()`|From `unicode-segmentation` crate|

---

## 📘 Crate Example

To count grapheme clusters:

```toml
# Cargo.toml
unicode-segmentation = "1.10.1"
```

```rust
use unicode_segmentation::UnicodeSegmentation;

fn main() {
    let s = "é"; // e + ◌́
    println!("Chars: {}", s.chars().count());        // 2
    println!("Graphemes: {}", s.graphemes(true).count()); // 1
}
```

---

## 🔍 Summary

|Concept|Description|Rust Support|
|---|---|---|
|**Unicode scalar value**|Individual code point (excluding surrogates)|`.chars()`|
|**Grapheme cluster**|User-visible character (may be multiple scalars)|`.graphemes()` via `unicode-segmentation`|
|**Why care?**|For proper string indexing, reversing, truncating, etc.|Helps avoid cutting characters in half|