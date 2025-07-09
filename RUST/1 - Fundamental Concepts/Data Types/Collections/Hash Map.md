---
tags: rust, datatype, collection, fundamental
---

- The type `HashMap<K, V>` stores a mapping of keys of type `K` to values of type `V` using a _hashing function_, which determines how it places these keys and values into memory.
- Hash maps are useful when you want to look up data not by using an index, as you can with vectors, but by using a key that can be of any type.

---

### 🔨 Creating a new Hash Map

- One way to create an empty hash map is to use `new` and to add elements with `insert`:
	```rust
	    use std::collections::HashMap;
	
	    let mut scores = HashMap::new();
	
	    scores.insert(String::from("Blue"), 10);
	    scores.insert(String::from("Yellow"), 50);
	```

- Note that we need to first `use` the `HashMap` from the collections portion of the standard library. Of our three common collections, this one is the least often used, so it’s not included in the features brought into scope automatically in the prelude. Hash maps also have less support from the standard library; there’s no built-in macro to construct them, for example.
- Just like vectors, hash maps store their data on the heap. This `HashMap` has keys of type `String` and values of type `i32`. 

```ad-important
Like vectors, hash maps are homogeneous: all of the keys must have the same type, and all of the values must have the same type.
```

---

### 👉 Accessing values in Hash Map

- We can get a value out of the hash map by providing its key to the `get` method:
	```rust
	    use std::collections::HashMap;
	
	    let mut scores = HashMap::new();
	
	    scores.insert(String::from("Blue"), 10);
	    scores.insert(String::from("Yellow"), 50);
	
	    let team_name = String::from("Blue");
	    let score = scores.get(&team_name).copied().unwrap_or(0);
	```
- We can iterate over each key-value pair in a hash map in a similar manner as we do with vectors, using a `for` loop:
	```rust
	    use std::collections::HashMap;
	
	    let mut scores = HashMap::new();
	
	    scores.insert(String::from("Blue"), 10);
	    scores.insert(String::from("Yellow"), 50);
	
	    for (key, value) in &scores {
	        println!("{key}: {value}");
	    }
	```

```ad-note
Iterating over a hash map happens in an arbitrary order.
```

---

### 🫳 Hash Map and Ownership

- For types that implement the `Copy` trait, like `i32`, the values are copied into the hash map. For owned values like `String`, the values will be moved and the hash map will be the owner of those values:
	```rust
	    use std::collections::HashMap;
	
	    let field_name = String::from("Favorite color");
	    let field_value = String::from("Blue");
	
	    let mut map = HashMap::new();
	    map.insert(field_name, field_value);
	    // field_name and field_value are invalid at this point, try using them and
	    // see what compiler error you get!
	```

```ad-note
If we insert references to values into the hash map, the values won’t be moved into the hash map. The values that the references point to must be valid for at least as long as the hash map is valid.
```

---

### ☝️ Updating a Hash Map

#### 1️⃣ Overwriting a Value
- If we insert a key and a value into a hash map and then insert that same key with a different value, the value associated with that key will be replaced:
	```rust
	    use std::collections::HashMap;
	
	    let mut scores = HashMap::new();
	
	    scores.insert(String::from("Blue"), 10);
	    scores.insert(String::from("Blue"), 25);
	
	    println!("{scores:?}"); // `{"Blue": 25}`
	```

#### 2️⃣ Adding a Key and Value Only If a Key Isn’t Present
- It’s common to check whether a particular key already exists in the hash map with a value and then to take the following actions: if the key does exist in the hash map, the existing value should remain the way it is; if the key doesn’t exist, insert it and a value for it.
- Hash maps have a special API for this called `entry` that takes the key you want to check as a parameter. The return value of the `entry` method is an enum called `Entry` that represents a value that might or might not exist.
	```rust
	    use std::collections::HashMap;
	
	    let mut scores = HashMap::new();
	    scores.insert(String::from("Blue"), 10);
	
	    scores.entry(String::from("Yellow")).or_insert(50); // returns a mutable reference
	    scores.entry(String::from("Blue")).or_insert(50);
	
	    println!("{scores:?}"); // `{"Yellow": 50, "Blue": 10}`
	```
- The `or_insert` method on `Entry` is defined to return a mutable reference to the value for the corresponding `Entry` key if that key exists, and if not, it inserts the parameter as the new value for this key and returns a mutable reference to the new value.

#### 3️⃣ Updating a Value Based on the Old Value

- Another common use case for hash maps is to look up a key’s value and then update it based on the old value:
	```rust
    use std::collections::HashMap;

    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    println!("{map:?}"); // `{"world": 2, "hello": 1, "wonderful": 1}`
	```

---

### #️⃣  Hashing Functions

- **Default Hasher**: Rust’s `HashMap` uses **SipHash** by default.
- **Why SipHash?**: It protects against **DoS attacks** by making it hard for attackers to cause hash collisions.
- **Trade-off**: SipHash is **secure but not the fastest**.
- **Need More Speed?**: You can use a **faster hasher** if performance is critical.
- **How?**: Provide a custom hasher that implements the `BuildHasher` trait.

👉 Use SipHash for safety by default, but swap it out if you **need speed and can accept less security**.