---
tags: rust, control, flow, fundamental
---

- The while keyword helps evaluate a condition within a loop. While a condition evaluates to `true`, the code runs; otherwise, it exits the loop.
	```rust
	fn main() {
	    let mut number = 3;
	
	    while number != 0 {
	        println!("{number}!");
	
	        number -= 1;
	    }
	
	    println!("LIFTOFF!!!");
	}
	```