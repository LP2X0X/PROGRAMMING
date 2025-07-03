---
tags: rust, control, flow, fundamental
---

### if ... let syntax

- The `if let` syntax lets you combine `if` and `let` into a less verbose way to handle values that [[match#^4a7317|match one pattern while ignoring the rest]].
	```rust
	    let config_max = Some(3u8);
	    if let Some(max) = config_max {
	        println!("The maximum is configured to be {max}");
	    }
	```

- We could also take advantage of the fact that expressions produce a value either to produce the `state` from the `if let` or to return early:
	```rust
	fn describe_state_quarter(coin: Coin) -> Option<String> {
	    let state = if let Coin::Quarter(state) = coin {
	        state
	    } else {
	        return None;
	    };
	
	    if state.existed_in(1900) {
	        Some(format!("{state:?} is pretty old, for America!"))
	    } else {
	        Some(format!("{state:?} is relatively new."))
	    }
	}
	```

- We can include an `else` with an `if let`. The block of code that goes with the `else` is the same as the block of code that would go with the `_` case in the `match` expression that is equivalent to the `if let` and `else`.
	```rust
	    let mut count = 0;
	    if let Coin::Quarter(state) = coin {
	        println!("State quarter from {state:?}!");
	    } else {
	        count += 1;
	    }
	```

---

### let ... else syntax

`let else` lets you **destructure values** with early exits if the pattern doesn't match — without writing a full `match` or `if let`.

It's perfect for cases where you want to:

- Destructure a value (`Option`, `Result`, struct, etc.)
    
- **Exit early** if it doesn't match
    

---

## ✅ Syntax

```rust
let Some(val) = maybe_val else {
    // This runs if the pattern doesn't match
    return; // or break, or panic, etc.
};
```

---

## 🧪 Example 1: Unwrapping `Option`

```rust
fn greet_user(user: Option<&str>) {
    let Some(name) = user else {
        println!("No user found.");
        return;
    };

    println!("Hello, {name}!");
}
```

> If `user` is `None`, the `else` block runs and returns early. Otherwise, `name` is available.

---

## 🧪 Example 2: Handling `Result`

```rust
fn divide(x: i32, y: i32) -> Result<i32, &'static str> {
    if y == 0 {
        return Err("division by zero");
    }
    Ok(x / y)
}

fn main() {
    let Ok(result) = divide(10, 2) else {
        println!("Failed to divide.");
        return;
    };

    println!("Result: {result}");
}
```

---

## 🧠 Why use `let else`?

- Cleaner than `if let` + `return`
    
- Removes nesting
    
- Makes early returns for invalid patterns concise and readable
    

---

## 🚫 Limitations

- You **must** return from the `else` block using `return`, `break`, `continue`, or `panic!`
    
- You **cannot** use it in `const` or `let` initializers yet
    
- Pattern must match **fully** — partial matching not supported
    

---

## 🧠 Summary

|Feature|`let else`|
|---|---|
|Purpose|Early return on pattern failure|
|Cleaner than|`if let` + `return`|
|Works with|`Option`, `Result`, custom enums|
|Introduced in|Rust 1.65|
