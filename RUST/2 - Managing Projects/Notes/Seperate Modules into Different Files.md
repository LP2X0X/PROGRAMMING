---
tags:
  - rust
  - note
---

- With a [[Module Tree|module tree]] like this:
	```pqsql
	crate
	├── system
	│   ├── io
	│   │   ├── reader
	│   │   └── writer
	│   └── network
	│       ├── client
	│       └── server
	├── engine
	│   ├── physics
	│   │   ├── collision
	│   │   └── dynamics
	│   └── audio
	│       ├── mixer
	│       └── effects
	```
- We can separate modules by declaring each child module in its parent file using `mod`, and placing the module’s definition in a file named after the child module.
- For example:
	- Filename: src/lib.rs
		```rust
		mod system;
		```
	- Filename: src/system.rs
		```rust
		// Implement "system" module in here without declare "system" module again
		mod io;
		```
	- Filename: src/system/io.rs 
		```rust
		// Implement "io" module in here without declare "io" module again
		```
