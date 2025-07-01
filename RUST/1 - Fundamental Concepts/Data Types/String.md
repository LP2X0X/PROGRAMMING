---
tags: rust, datatype, fundamental
---

- The **String** type manages data allocated on the heap and as such is able to store an amount of text unknown at compile time.
- You can create a `String` from a string literal using the `from` function, like so:
	```rust
	let s = String::from("hello");
	```

- This kind of string _can_ be mutated:
	```rust
	    let mut s = String::from("hello");
	
	    s.push_str(", world!"); // push_str() appends a literal to a String
	
	    println!("{s}"); // This will print `hello, world!`
	```