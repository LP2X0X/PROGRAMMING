---
tags: rust, datatype, string, fundamental
---

- A _string slice_ is a [[RUST/1 - Fundamental Concepts/Terms/Reference|reference]] to part of a `String`, the type that signifies “string slice” is written as `&str`, and it looks like this:
	```rust
	let s = String::from("hello world");

	let hello = &s[0..5];
	let hello_different_syntax = &s[..5];
	
	let world = &s[6..11];
	let world_different_syntax = &s[6..];
	
	let hello_world = &s[..];

	// Function example
	fn first_word(s: &String) -> &str {
		let bytes = s.as_bytes();
	
		for (i, &item) in bytes.iter().enumerate() {
			if item == b' ' {
				return &s[0..i];
			}
		}
	
		&s[..]
	}
	```

```ad-important
A **string slice** (`&str`) accesses the **bytes** of the string — **not characters**.
```

- We create slices using a range within brackets by specifying `[starting_index..ending_index]`:
	- Where _`starting_index`_ is the first position in the slice
	- And _`ending_index`_ is <u>one more</u> than the last position in the slice

```ad-note
With Rust’s `..` range syntax, if you want to start at index 0, you can drop the value before the two periods. By the same token, if your slice includes the last byte of the `String`, you can drop the trailing number. You can also drop both values to take a slice of the entire string.
```

- Internally, the slice data structure stores the starting position and the length of the slice, which corresponds to _`ending_index`_ minus _`starting_index`_.

```ad-warning
String slice range indices must occur at valid UTF-8 character boundaries.
```