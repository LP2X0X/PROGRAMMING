### Type deduction drops const

- Type deduction will drop `const` from types:

```cpp
int main()
{
    const double a { 7.8 }; // a has type const double
    auto b { a };           // b has type double (const dropped)

    constexpr double c { 7.8 }; // c has type const double (constexpr implicitly applies const)
    auto d { c };               // d has type double (const dropped)

    return 0;
}
```

- Const (or constexpr) can be applied by adding the `const` (or `constexpr`) qualifier in the definition:

```cpp
int main()
{
    double a { 7.8 };    // a has type double
    const auto b { a };  // b has type const double (const applied)

    constexpr double c { 7.8 }; // c has type const double (constexpr implicitly applies const)
    const auto d { c };         // d is const double (const dropped, const reapplied)
    constexpr auto e { c };     // e is constexpr double (const dropped, constexpr reapplied)

    return 0;
}
```

---

### Type deduction drops references

- In addition to dropping const, type deduction will also drop references:

```cpp
#include <string>

std::string& getRef(); // some function that returns a reference

int main()
{
    auto ref { getRef() }; // type deduced as std::string (not std::string&)

    return 0;
}
```

- In the above example, variable `ref` is using type deduction. Although function `getRef()` returns a `std::string&`, the reference qualifier is dropped, so the type of `ref` is deduced as `std::string`.

- Just like with dropped const, if you want the deduced type to be a reference, you can reapply the reference at the point of definition:

```cpp
#include <string>

std::string& getRef(); // some function that returns a reference

int main()
{
    auto ref1 { getRef() };  // std::string (reference dropped)
    auto& ref2 { getRef() }; // std::string& (reference dropped, reference reapplied)

    return 0;
}
```

---

### Type deduction and const references

- If the initializer is a reference to const, the reference is dropped first (and then reapplied if applicable), and then any top-level const is dropped from the result.

```cpp
#include <string>

const std::string& getConstRef(); // some function that returns a reference to const

int main()
{
    auto ref1{ getConstRef() }; // std::string (reference dropped, then top-level const dropped from result)

    return 0;
}
```

- In the above example, since `getConstRef()` returns a `const std::string&`, the reference is dropped first, leaving us with a `const std::string`. This const is now a top-level const, so it is also dropped, leaving the deduced type as `std::string`.
- Dropping a reference may change a low-level const to a top-level const: `const std::string&` is a low-level const, but dropping the reference yields `const std::string`, which is a top-level const.
  
---

### Type deduction and pointers

- Unlike references, type deduction does not drop pointers:

```cpp
#include <string>

std::string* getPtr(); // some function that returns a pointer

int main()
{
    auto ptr1{ getPtr() }; // std::string*

    return 0;
}
```