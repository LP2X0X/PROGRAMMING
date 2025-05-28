---
tags: cpp, term, fundamental
---

`using` is a **C++ language keyword**, specifically part of the **declarative syntax** used during the **compilation phase**, not preprocessing.

```ad-important
using is not a [[Preprocessor Directive|preprocessor directive]].
```

---

### 📌 Types of `using` in C++:

1. **Namespace Importing**

```cpp
using namespace std;
```
- This tells the compiler that unqualified names should be looked up in the `std` namespace.
- It’s resolved **at compile time**, not preprocessed.

2. **Type Alias (C++11 onward)**

```cpp
using Vec = std::vector<int>;
```
- A cleaner, more powerful alternative to `typedef`.
- Defines a type alias directly in code.

3. **Base Class Member Declaration**

```cpp
class Derived : public Base {
	using Base::someFunction;
};
```
- Makes a member function or type from the base class visible in the derived class.

---

### 🧠 Summary

- `using` is **not a directive** at all — it's a **C++ keyword**.
- It plays a role in **namespaces**, **type aliases**, and **inheritance visibility**.
- It is handled **by the compiler**, **not** by the C preprocessor.