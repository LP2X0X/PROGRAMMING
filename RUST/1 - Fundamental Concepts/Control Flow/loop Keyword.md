---
tags: rust, control, flow, fundamental
---

- The `loop` keyword tells Rust to execute a block of code over and over again forever or until you explicitly tell it to stop.
	- You can place the `break` keyword within the loop to tell the program when to stop executing the loop.
	- You can also use `continue` which in a loop tells the program to skip over any remaining code in this iteration of the loop and go to the next iteration.

	```rust
	fn main() {
	    loop {
	        println!("again!");
	    }
	}
	```

- To pass the result of an operation out of the loop to the rest of your code, you can add the value you want returned after the `break` expression you use to stop the loop; that value will be returned out of the loop so you can use it:
	```rust
	fn main() {
	    let mut counter = 0;
	
	    let result = loop {
	        counter += 1;
	
	        if counter == 10 {
	            break counter * 2;
	        }
	    };
	
	    println!("The result is {result}");
	}
	```

- You can optionally specify a _loop label_ on a loop that you can then use with `break` or `continue` to specify that those keywords apply to the labeled loop instead of the innermost loop. 
	- Loop labels must **begin** with a single quote:
	```rust
	fn main() {
		let mut count = 0;
		'counting_up: loop {
			println!("count = {count}");
			let mut remaining = 10;
	
			loop {
				println!("remaining = {remaining}");
				if remaining == 9 {
					break; // The first `break` that doesn’t specify a label will exit the inner loop only.
				}
				if count == 2 {
					break 'counting_up;
				}
				remaining -= 1;
			}
	
			count += 1;
		}
		println!("End count = {count}");
	}
	```

- You can use a `for` loop and execute some code for each item in a collection:
	```rust
	fn main() {
	    let a = [10, 20, 30, 40, 50];
	
	    for element in a {
	        println!("the value is: {element}");
	    }
	}
	```