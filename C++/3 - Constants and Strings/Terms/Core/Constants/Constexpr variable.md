---
tags:
  - cpp
  - term
  - fundamental
---

- The `constexpr` keyword (which is shorthand for “constant expression”) can enlist the compiler’s help to ensure we get a compile-time constant variable where we desire one. A **constexpr variable** is always a compile-time constant. As a result, a constexpr variable must be initialized with a [[Constant expression|constant expression]], otherwise a compilation error will result.

```cpp
#include <iostream>

// The return value of a non-constexpr function is not constexpr
int five()
{
    return 5;
}

int main()
{
    constexpr double gravity { 9.8 }; // ok: 9.8 is a constant expression
    constexpr int sum { 4 + 5 };      // ok: 4 + 5 is a constant expression
    constexpr int something { sum };  // ok: sum is a constant expression

    std::cout << "Enter your age: ";
    int age{};
    std::cin >> age;

    constexpr int myAge { age };      // compile error: age is not a constant expression
    constexpr int f { five() };       // compile error: return value of five() is not constexpr

    return 0;
}
```

```ad-important
Additionally, `constexpr` works for variables with non-integral types.
```
