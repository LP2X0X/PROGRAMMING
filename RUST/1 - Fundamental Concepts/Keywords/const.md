---
tags:
  - rust
  - keyword
  - fundamental
---

- Constants aren’t just immutable by default—they’re always immutable. You declare constants using the `const` keyword instead of the `let` keyword, and the type of the value _must_ be annotated.
- Constants can be declared in any scope, including the global scope, which makes them useful for values that many parts of code need to know about.
- Constants may be set only to a constant expression, not the result of a value that could only be computed at runtime.

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

```ad-note
Constants are valid for the entire time a program runs, within the scope in which they were declared.
```