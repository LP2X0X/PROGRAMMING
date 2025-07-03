---
tags: rust, term
---

The **prelude** in Rust is a collection of **commonly used items** (traits, types, macros) that are **automatically imported** into every Rust file — so you can use them **without needing to explicitly `use` them**.

---

## 📦 What is in the Rust Prelude?

The Rust standard library defines a **prelude module**, and every Rust file gets:

```rust
use std::prelude::v1::*;
```

automatically (unless you're in `#![no_std]` or using a custom prelude).

---

### ✅ Most Important Prelude Items

#### 🔹 Common Types

|Type|Description|
|---|---|
|`Option<T>`|Represents `Some` or `None`|
|`Result<T, E>`|Used for error handling|
|`String`, `ToString`|Heap-allocated strings|
|`Vec<T>`|Growable array type|
|`Box<T>`|Smart pointer for heap allocation|

#### 🔹 Common Traits

|Trait|Used For|
|---|---|
|`Copy`, `Clone`|Copying and cloning values|
|`Debug`|`{:?}` formatting|
|`Default`|Provides `default()` method|
|`PartialEq`, `Eq`|Equality comparisons|
|`PartialOrd`, `Ord`|Ordering comparisons|
|`AsRef`, `AsMut`|Conversion to reference types|
|`From`, `Into`|Type conversion|
|`Iterator`|Iteration (`for` loops, `.map()`, etc.)|

#### 🔹 Macros

|Macro|Description|
|---|---|
|`println!`, `print!`|Print to stdout|
|`format!`|Create formatted strings|
|`vec!`|Create a `Vec<T>`|
|`dbg!`|Debug print with file/line|
|`todo!`, `unimplemented!`|Placeholder panic macros|

---

## 📌 Example: These work without explicit imports

```rust
fn greet(name: &str) -> Option<String> {
    if name.is_empty() {
        None
    } else {
        Some(format!("Hello, {}!", name))
    }
}
```

No need to write:

```rust
use std::option::Option;
use std::format;
```

Because they're in the **prelude**.

---

## 🚫 Not in the Prelude

Some frequently used types **are not in the prelude** and must be imported manually:

- `HashMap`, `HashSet`: `use std::collections::HashMap;`
    
- `Rc`, `Arc`, `Cell`, `RefCell`: `use std::rc::Rc;`, etc.
    
- `fs`, `io`, `thread`, `time`: Manual imports needed
    

---

## 🛠 Custom Prelude?

You can make your own mini-prelude by creating a `mod prelude` and importing it everywhere:

```rust
// prelude.rs
pub use std::collections::*;
pub use std::sync::*;
```

Then in your modules:

```rust
use crate::prelude::*;
```