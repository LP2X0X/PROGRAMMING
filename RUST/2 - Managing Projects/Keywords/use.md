---
tags:
  - rust
  - keyword
  - fundamental
---

In Rust, the `use` keyword is used to **bring items (functions, structs, modules, traits, etc.) into scope** — kind of like `import` in other languages, but with more precise control.

```ad-note
`use` only creates the shortcut for the particular scope in which the `use` occurs.
```

```ad-note
The child module can reference the shortcut in the parent module with `super::<ref_mod_name>`.
```

---

## ✅ Basic Syntax

```rust
use path::to::item;
```

This allows you to **use the item without writing its full path** every time.

---

## 🧠 Why use `use`?

Without `use`:

```rust
let now = std::time::SystemTime::now();
```

With `use`:

```rust
use std::time::SystemTime;

let now = SystemTime::now();
```

More readable, less repetitive.

---

## 🔹 Example: Bringing a struct into scope

```rust
use std::collections::HashMap;

let mut map = HashMap::new();
```

---

## 🔹 Example: Bringing a function into scope

- Bringing the function’s parent module into scope with `use` means we have to specify the parent module when calling the function.
- Specifying the parent module when calling the function makes it clear that the function isn’t locally defined while still minimizing repetition of the full path.

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

---

## 🔸 Bring multiple items from the same module

- We can use nested paths to bring the same items into scope in one line.

```rust
use std::cmp::{PartialEq, Ordering};
```

```rust
use std::io;
use std::io::Write;

// We can use this
use std::io::{self, Write};
```

---

## 🔸 Bring all public items from a module (glob import)

- The glob operator is often used when testing to bring everything under test into the `tests` module.

```rust
use std::io::*;
```

⚠️ Use with caution — this can make it unclear where things come from.

---

## 🔸 Rename on import (to avoid conflict or shorten)

```rust
use std::fmt::Result as FmtResult;
```

---

## 🔸 `use` with nested modules

```rust
mod utils {
    pub fn greet() {
        println!("Hello!");
    }
}

use utils::greet;

fn main() {
    greet();
}
```

---

## 🔹 Use `use` inside modules

You can declare `use` inside modules to access code cleanly:

```rust
mod config {
    use crate::utils::greet;

    pub fn init() {
        greet();
    }
}
```

---

## 🔸 Re-export with `pub use`

If you want to expose something from your module to others:

```rust
pub use crate::utils::greet;
```

Now external code can do:

```rust
mycrate::greet();
```

---

## 🧠 Summary Table

|Usage|Meaning|
|---|---|
|`use path::to::item;`|Bring a single item into scope|
|`use path::{A, B};`|Import multiple items|
|`use path::*;`|Glob import (all public items)|
|`use path::item as NewName;`|Rename on import|
|`pub use path::item;`|Re-export the item from this module|
