---
tags:
  - rust
  - keyword
  - fundamental
---

- There’s another solution to the problem of bringing two types of the same name into the same scope with `use`: after the path, we can specify `as` and a new local name, or _alias_, for the type.
	```rust
	use std::fmt::Result;
	use std::io::Result as IoResult;
	
	fn function1() -> Result {
	    // --snip--
	}
	
	fn function2() -> IoResult<()> {
	    // --snip--
	}
	```