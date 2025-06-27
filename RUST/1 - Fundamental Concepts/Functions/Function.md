---
tags: rust, fundamental
---

- We define a function in Rust by entering `fn` followed by a function name and a set of parentheses. The curly brackets tell the compiler where the function body begins and ends.
	- Rust code uses _snake case_ as the conventional style for function and variable names, in which all letters are lowercase and underscores separate words.
	- Rust doesn’t care where you define your functions, only that they’re defined somewhere in a scope that can be seen by the caller.
	```rust
	fn main() {
	    println!("Hello, world!");
	
	    another_function();
	}
	
	fn another_function() {
	    println!("Another function.");
	}
	```

- Functions can return values to the code that calls them.
	- We don’t name return values, but we must declare their type after an arrow (`->`). 
	- In Rust, the return value of the function is synonymous with the value of the final expression in the block of the body of a function. You can return early from a function by using the `return` keyword and specifying a value, but most functions *return the last expression implicitly*.
	```rust
	fn five() -> i32 {
	    5
	}
	
	fn main() {
	    let x = five();
	
	    println!("The value of x is: {x}");
	}
	```