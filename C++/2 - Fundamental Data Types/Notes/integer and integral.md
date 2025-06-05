---
tags: cpp, distinguish, fundamental
---

- In mathematics, an “integer” is a number with no decimal or fractional part, including negative and positive numbers and zero. The term “integral” has several different meanings, but in the context of C++ is used to mean “like an integer”.

- The C++ standard defines the following terms:
	- The **standard integer types** are `short`, `int`, `long`, `long long` (including their signed and unsigned variants).
	- The **integral types** are `bool`, the various char types, and the standard integer types.

```ad-note
All integral types are stored in memory as integer values, but only the standard integer types will display as an integer value when output. 
```

- The C++ standard also explicitly notes that “integer types” is a synonym for “integral types”. However, conventionally, “integer types” is more often used as shorthand for the “standard integer types” instead.

```ad-note
Note that the term “integral types” only includes fundamental types. This means non-fundamental types (such as `enum` and `enum class`) are not integral types, even when they are stored as an integer (and in the case of `enum`, displayed as one too).
```
