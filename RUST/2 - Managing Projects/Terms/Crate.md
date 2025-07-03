---
tags: rust, term, project_management, fundamental
---

- A _crate_ is the smallest amount of code that the Rust compiler considers at a time.
- A crate can come in one of two forms: a binary crate or a library crate.
	- _Binary crates_ are programs you can compile to an executable that you can run, such as a command line program or a server. Each must have a function called `main` that defines what happens when the executable runs. All the crates we’ve created so far have been binary crates.
	- _Library crates_ don’t have a `main` function, and they don’t compile to an executable. Instead, they define functionality intended to be shared with multiple projects. Most of the time when Rustaceans say “crate”, they mean library crate, and they use “crate” interchangeably with the general programming concept of a “library”.
		- A package can have multiple binary crates by placing files in the _src/bin_ directory: each file will be a separate binary crate.