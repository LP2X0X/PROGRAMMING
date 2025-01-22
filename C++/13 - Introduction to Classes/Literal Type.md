- In C++, a **literal type** is any type for which it might be possible to create an object within a constant expression. Put another way, an object can’t be constexpr unless the type qualifies as a literal type. And our non-aggregate `Pair` does not qualify.

```ad-note
A literal and a literal type are distinct (but related) things. A literal is a constexpr value that is inserted into the source code. A literal type is a type that can be used as the type of a constexpr value. A literal always has a literal type. However, a value or object with a literal type need not be a literal.
```

- The definition of a literal type is complex, and a summary can be found on [cppreference](https://en.cppreference.com/w/cpp/named_req/LiteralType). However, it’s worth noting that literal types include:
	- Scalar types (those holding a single value, such as fundamental types and pointers)
	- Reference types
	- Most aggregates
	- Classes that have a constexpr constructor

```ad-tip
Best practice
If you want your class to be able to be evaluated at compile-time, make your member functions and constructor constexpr.
```
