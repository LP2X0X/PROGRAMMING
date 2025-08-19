---
tags:
  - rust
  - term
  - project_management
  - fundamental
---

- A _package_ is a bundle of one or more [[Crate|crates]] that provides a set of functionality. A package contains a _Cargo.toml_ file that describes how to build those crates.
- A package can contain as many binary crates as you like, but at most only one library crate. A package must contain at least one crate, whether that’s a library or binary crate.