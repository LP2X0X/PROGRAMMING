---
tags: rust, datatype, fundamental
---

- Another way to have a collection of multiple values is with an _array_. Unlike a tuple, every element of an array **must have the same type**.
	- Unlike arrays in some other languages, arrays in Rust *have a fixed length*.
	
- You write an array’s type using square brackets with the type of each element, a semicolon, and then the number of elements in the array, like so:
	```rust
	let a: [i32; 5] = [1, 2, 3, 4, 5];
	```

- Arrays are useful when you want your data allocated on the stack, the same as the other types we have seen so far, rather than the heap or when you want to ensure you always have a fixed number of elements.

</br>

- You can also initialize an array to contain the same value for each element by specifying the initial value, followed by a semicolon, and then the length of the array in square brackets, as shown here:
	```rust
	let a = [3; 5];
	```

</br>

- An array is a single chunk of memory of a known, fixed size that can be allocated on the stack. You can access elements of an array using indexing, like this:
	```rust
	fn main() {
	    let a = [1, 2, 3, 4, 5];
	
	    let first = a[0];
	    let second = a[1];
	}
	```