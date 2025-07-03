---
tags: rust, term, project_management, fundamental
---

## 🌳 What Is a Module Tree?

In Rust, a **module tree** is the hierarchy of **modules** ([[RUST/2 - Managing Projects/Terms/Module|mod]]) that organize your code.  
It reflects the structure of your source files and folders.

Think of it like:

```
crate
├── mod1
│   └── mod2
└── mod3
```

Each `mod` = one node in the tree.

---

## 🛣️ Paths

To show Rust where to find an item in a module tree, we use a path in the same way we use a path when navigating a filesystem. To call a function, we need to know its path.

- A path can take two forms:
	- An _absolute path_ is the full path starting from a crate root; for code from an external crate, the absolute path begins with the crate name, and for code from the current crate, it starts with the literal `crate`.
	- A _relative path_ starts from the current module and uses `self`, `super`, or an identifier in the current module.

Both absolute and relative paths are followed by one or more identifiers separated by double colons (`::`).

---

## ✅ Example: Simple Module Tree

### File: `src/lib.rs`

```rust
pub mod utils;
mod internal;
```

- `utils` is public (can be used outside)
    
- `internal` is private (used only inside crate)
    

---

### File: `src/utils.rs`

```rust
pub fn greet(name: &str) {
    println!("Hello, {name}!");
}
```

Now, you can use:

```rust
use mycrate::utils::greet;
```

---

### File: `src/internal.rs`

```rust
pub fn hidden() {
    println!("Internal use only");
}
```

Accessible inside the crate only.

---

## 📁 Nested Module Tree (with folders)

If you want modules inside folders, do this:

### File: `src/network/mod.rs`

```rust
pub mod tcp;
pub mod udp;
```

### Files:

- `src/network/tcp.rs`
    
- `src/network/udp.rs`
    

Now your tree looks like:

```
crate
└── network
    ├── tcp
    └── udp
```

You can access:

```rust
use crate::network::tcp::connect;
```

---

## 📌 Rules of Rust Modules

|Rule|Description|
|---|---|
|`mod something;`|Declares a submodule (loads `something.rs` or `something/mod.rs`)|
|Folder structure|Must mirror module structure|
|`pub` needed|To expose modules/functions outside their parent|
|`use` needed|To bring items into scope|

---

## 🔍 Real-World Layout

```
src/
├── lib.rs           # entry point for library crate
├── main.rs          # (if binary crate)
├── utils.rs
├── config/
│   ├── mod.rs       # declares config module
│   └── loader.rs    # config::loader
│   └── parser.rs    # config::parser
```

In `lib.rs`:

```rust
pub mod utils;
pub mod config;
```

In `config/mod.rs`:

```rust
pub mod loader;
pub mod parser;
```

---

## 🧠 Summary

|Concept|Example|
|---|---|
|Declare module|`mod foo;`|
|Submodules|`mod bar;` inside `foo.rs` or `foo/mod.rs`|
|Access items|`crate::foo::bar::baz()`|
|Public items|Use `pub` to expose|
