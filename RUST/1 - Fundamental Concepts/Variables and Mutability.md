---
tags: rust, fundamental
---

- By default, variables in Rust are immutable. 

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {x}");
    x = 6; // This will cause compile time error
    println!("The value of x is: {x}");
}
```