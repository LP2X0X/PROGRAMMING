---
tags:
  - rust
  - datatype
  - scalar
  - fundamental
---

- You can write integer literals in any of the forms shown in the table below. 
- Note that number literals that can be multiple numeric types allow a type suffix, such as `57u8`, to designate the type. Number literals can also use `_` as a visual separator to make the number easier to read, such as `1_000`, which will have the same value as if you had specified `1000`.

| Number literals  | Example       |
| ---------------- | ------------- |
| Decimal          | `98_222`      |
| Hex              | `0xff`        |
| Octal            | `0o77`        |
| Binary           | `0b1111_0000` |
| Byte (`u8` only) | `b'A'`        |

---

## 🧮 Integer Type Suffixes

|Suffix|Type|Signed|Size|Range|
|---|---|---|---|---|
|`i8`|`i8`|Yes|8-bit|–128 to 127|
|`i16`|`i16`|Yes|16-bit|–32,768 to 32,767|
|`i32`|`i32`|Yes|32-bit|–2,147,483,648 to 2,147,483,647|
|`i64`|`i64`|Yes|64-bit|–9 quintillion to +9 quintillion|
|`i128`|`i128`|Yes|128-bit|Massive range|
|`isize`|`isize`|Yes|pointer-sized|Depends on system arch|

---

|Suffix|Type|Signed|Size|Range|
|---|---|---|---|---|
|`u8`|`u8`|No|8-bit|0 to 255|
|`u16`|`u16`|No|16-bit|0 to 65,535|
|`u32`|`u32`|No|32-bit|0 to 4.2 billion|
|`u64`|`u64`|No|64-bit|0 to 18 quintillion|
|`u128`|`u128`|No|128-bit|Extremely large range|
|`usize`|`usize`|No|pointer-sized|Depends on system arch|

---

## 🔢 Floating-Point Type Suffixes

|Suffix|Type|Size|Precision|
|---|---|---|---|
|`f32`|`f32`|32-bit|~6 decimal digits|
|`f64`|`f64`|64-bit|~15 decimal digits|

---

## 📝 Examples

```rust
let a = 100u8;     // Unsigned 8-bit
let b = -42i16;    // Signed 16-bit
let c = 3.14f32;   // 32-bit float
let d = 1_000_000usize; // usize (arch dependent)
```

---

## 📌 Default Types (if you don't write a suffix)

|Kind|Default Type|
|---|---|
|Integer|`i32`|
|Float|`f64`|

Example:

```rust
let x = 5;     // i32 by default
let y = 3.14;  // f64 by default
```
