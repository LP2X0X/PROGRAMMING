---
tags: [design, pattern, behavioral, iterator]
---

## Overview

Provides a way to **access elements of a collection sequentially** without exposing its underlying representation (array, linked list, tree, etc.).

## When to Use

- When you want to traverse a collection without exposing its internals
- When you need multiple traversal algorithms over the same collection

```ad-note
In C++, the STL already implements the Iterator pattern extensively via `begin()`, `end()`, and range-based for loops. You rarely need to implement this from scratch in modern C++.
```
