---
tags:
  - rust
  - datatype
  - collection
  - fundamental
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

    let third: &i32 = &v[2]; // Indexing
    println!("The third element is {third}");

    let third: Option<&i32> = v.get(2); // get method
    match third {
        Some(third) => println!("The third element is {third}"),
        None => println!("There is no third element."),
    }
```

---

### Iterating over a Vector

- To access each element in a vector in turn, we would iterate through all of the elements rather than use indices to access one at a time.
	```rust
	let v = vec![100, 32, 57];
	for i in &v {
		println!("{i}");
	}
	```
- We can also iterate over mutable references to each element in a mutable vector in order to make changes to all the elements.
	```rust
	let mut v = vec![100, 32, 57];
	    for i in &mut v {
	        *i += 50;
	    }
	```

---

### Using an Enum to Store Multiple Types

- In Rust, vectors can only store elements of the **same type**, which can be limiting when you want to store **multiple types** together.
- To work around this, you can define an **enum** with variants for each type you want to store. Since all enum variants share the same type (the enum), you can store them together in a vector.
	```rust
	enum SpreadsheetCell {
	    Int(i32),
	    Float(f64),
	    Text(String),
	}
	
	let row = vec![
	    SpreadsheetCell::Int(3),
	    SpreadsheetCell::Text(String::from("blue")),
	    SpreadsheetCell::Float(10.12),
	];
	```

- Rust needs to know the types at compile time for memory safety and performance. Using enums ensures **type safety** and forces you to handle all possible cases (via `match`).