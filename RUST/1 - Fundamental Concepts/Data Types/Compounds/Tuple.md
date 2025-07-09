---
tags: rust, datatype, compound, fundamental
---

- A _tuple_ is a general way of grouping together a number of values with a variety of types into one compound type.
	- Tuples have a fixed length: once declared, they cannot grow or shrink in size.

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1); // different types in same tuple
}
```

- To get the individual values out of a tuple, we can use pattern matching to **destructure** a tuple value, like this:
	```rust
	fn main() {
	    let tup = (500, 6.4, 1);
	
	    let (x, y, z) = tup;
	
	    println!("The value of y is: {y}");
	}
	```

- We can also **access** a tuple element directly by using a period (`.`) followed by the index of the value we want to access. For example:
	```rust
	fn main() {
	    let x: (i32, f64, u8) = (500, 6.4, 1);
	
	    let five_hundred = x.0;
	
	    let six_point_four = x.1;
	
	    let one = x.2;
	}
	```