---
tags: rust, fundamental
---

- We can define functions to have _parameters_, which are special variables that are part of a function’s signature.
	```rust
	fn print_labeled_measurement(value: i32, unit_label: char) {
	    println!("The measurement is: {value}{unit_label}");
	}
	```