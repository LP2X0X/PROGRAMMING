---
tags:
  - rust
  - file_extension
  - fundamental
---

## **📄 What is Cargo.toml ?**

Cargo.toml is the **manifest file** for your Rust project. It describes:

- your package metadata (name, version, authors),
- dependencies,
- build settings,
- features,
- and more.

It uses the [TOML format](https://toml.io/en/) (Tom’s Obvious, Minimal Language), which is like a more readable version of INI or JSON.

---

## **🧱 Basic Structure of Cargo.toml**

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <you@example.com>"]
description = "A simple example project"
license = "MIT"

[dependencies]
rand = "0.8"
```

---

## **🧩 Sections Explained**

### **\[package]**


Defines general info about your crate (required):

- name: The name of your package.
- version: Follows semantic versioning.
- edition: Rust language edition (e.g., 2018, 2021, or 2024).
- Other optional fields: description, license, homepage, repository, etc.

---

### **\[dependencies]**

Lists **external crates** your project depends on:

```toml
serde = "1.0"
rand = "0.8"
```

You can also specify versions, features, and sources:

```toml
serde = { version = "1.0", features = ["derive"] }
```

Or use Git/local paths:

```toml
my_crate = { git = "https://github.com/user/my_crate" }
my_local = { path = "../my_local_crate" }
```

---

### **\[dev-dependencies]**

Used **only during development** (e.g., for tests, benchmarks):

```toml
pretty_assertions = "1.0"
```

---

### **\[build-dependencies]**

Dependencies required only for a **build script** (build.rs):

```toml
cc = "1.0"
```

---

### **\[features]**

Optional parts of your crate users can enable:

```toml
[features]
default = ["json"]
json = ["serde_json"]
```

---

### **\[workspace]**

For **multi-crate projects**, this section defines the workspace and its members:

```toml
[workspace]
members = ["core", "cli"]
```

---

## **🔍 Cargo.lock**

In addition to Cargo.toml, Cargo generates Cargo.lock:

- Records **exact versions** of dependencies used.
- Ensures reproducible builds.
- Should be committed for binaries, but not for libraries.

---

## **✅ Real Example**

```toml
[package]
name = "timevault"
version = "0.1.0"
edition = "2021"

[dependencies]
chrono = "0.4"
serde = { version = "1.0", features = ["derive"] }
```

This config:

- Defines the project name and Rust edition.
- Adds dependencies on chrono and serde with the derive feature enabled.

---

## **💡 Tip**

You can use cargo add crate_name (from cargo-edit) to automatically update your Cargo.toml without editing it manually.