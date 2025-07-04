---
tags: rust, collection, fundamental
---

- Vectors (`Vec<T>`) allow you to store more than one value in a single data structure that puts all the values next to each other in memory. Vectors can only store values of the same type.

---

### Create a new Vector

- To create a new empty vector, we call the `Vec::new` function, as shown in Listing 8-1.
	```rust
	    let v: Vec<i32> = Vec::new();
	```
- Rust conveniently provides the `vec!` macro, which will create a new vector that holds the values you give it.
	```rust
	    let v = vec![1, 2, 3];
	```

---

### Updating a Vector

- To create a vector and then add elements to it, we can use the `push` method:
	```rust
	    let mut v = Vec::new();
	
	    v.push(5);
	    v.push(6);
	    v.push(7);
	    v.push(8);
	```

---

### Reading element of Vector

- There are two ways to reference a value stored in a vector: 
	- Via indexing.
	- Using the `get` method.

```rust
    let v = vec![1, 2, 3, 4, 5];

    let third: &i32 = &v[2];
    println!("The third element is {third}");

    let third: Option<&i32> = v.get(2);
    match third {
        Some(third) => println!("The third element is {third}"),
        None => println!("There is no third element."),
    }
```