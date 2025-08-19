---
tags:
  - js
  - term
  - fundamental
---

- Any object that isn’t created with an explicit constructor function is instead implicitly created with JavaScript’s built-in Object constructor function.
- The prototype this constructor references, available as **Object.prototype**, contains basic methods that all objects should inherit.
- This prototype object marks the end of the line in the chain of prototype references. All objects eventually trace their origin back to Object.prototype.
