---
tags: rust, keyword, fundamental
---

- Although variables are immutable by default, you can make them mutable by adding `mut` in front of the variable name. Adding `mut` also conveys intent to future readers of the code by indicating that other parts of the code will be changing this variable’s value.
  
```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {x}");
    x = 6; // This won't cause a compile time error
    println!("The value of x is: {x}");
}
```