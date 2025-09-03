---
tags:
  - js
  - note
  - fundamental
---

- The important difference between [[OR| OR operator]] and [[null coalescing operator]]  is that:
	- `||` returns the first _truthy_ value.
	- `??` returns the first _defined_ value.
- In other words, `||` doesn’t distinguish between `false`, `0`, an empty string `""` and `null/undefined`. They are all the same – falsy values. If any of these is the first argument of `||`, then we’ll get the second argument as the result.