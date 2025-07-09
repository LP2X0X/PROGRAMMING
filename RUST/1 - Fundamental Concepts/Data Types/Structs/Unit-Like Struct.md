---
tags: rust, datatype, struct, fundamental
---

- In Rust, an **unit-like struct** is a struct that don'y have any fields. It behave similarly to (), the [[Unit|unit]] type.
	```rust
	struct AlwaysEqual;
	
	fn main() {
	    let subject = AlwaysEqual;
	}
	```