---
tags: rust, keyword, fundamental
---

- The primary use for the `let` keyword is in `let` statements, which are used to introduce a new set of variables into the current scope, as given by a pattern.
	- By using `let`, we can perform a few transformations on a value but have the variable be immutable after those transformations have been completed.
	- Using the `let` keyword again, we can change the type of the value but reuse the same name with the help of [[Shadowing|shadowing]].

```rust
let spaces = "   ";
let spaces = spaces.len();
```