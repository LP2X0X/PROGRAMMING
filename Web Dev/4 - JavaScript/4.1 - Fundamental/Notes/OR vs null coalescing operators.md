---
tags:
  - js
  - note
  - fundamental
---

- The important difference between [[Web Dev/4 - JavaScript/4.1 - Fundamental/4.1.3 - Operators/Logical Operators/OR| OR operator]] and [[nullish coalescing operator]]  is that:
	- `||` returns the first _truthy_ value.
	- `??` returns the first _defined_ value.
- In other words, `||` doesn’t distinguish between `false`, `0`, an empty string `""` and `null/undefined`. They are all the same – falsy values. If any of these is the first argument of `||`, then we’ll get the second argument as the result.