---
tags: rust, control, flow, fundamental
---

- All `if` expressions start with the keyword `if`, followed by a condition.
	```rust
	fn main() {
	    let number = 3;
	
	    if number < 5 {
	        println!("condition was true");
	    } else {
	        println!("condition was false");
	    }
	}
	```
- Unlike languages such as Ruby and JavaScript, Rust will not automatically try to convert non-Boolean types to a Boolean. You must be explicit and always provide `if` with a Boolean as its condition.

</br>

- Because `if` is an expression, we can use it on the right side of a `let` statement to assign the outcome to a variable:
	```rust
	fn main() {
	    let condition = true;
	    let number = if condition { 5 } else { 6 };
	
	    println!("The value of number is: {number}");
	}
	```
- In this case, the value of the whole `if` expression depends on which block of code executes. This means the values that have the potential to be results from each arm of the `if` **must be the same type**.
