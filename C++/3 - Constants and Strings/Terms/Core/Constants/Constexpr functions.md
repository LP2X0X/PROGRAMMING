---
tags: cpp, term, fundamental
---

- A **constexpr function** is a function that is allowed to be called in a [[Constant expression|constant expression]]. If a required constant expression contains a constexpr function call, that constexpr function call must evaluate at compile-time.

```cpp
#include <iostream>

int max(int x, int y) // this is a non-constexpr function
{
    if (x > y)
        return x;
    else
        return y;
}

constexpr int cmax(int x, int y) // this is a constexpr function
{
    if (x > y)
        return x;
    else
        return y;
}

int main()
{
    int m1 { max(5, 6) };            // ok
    const int m2 { max(5, 6) };      // ok
    constexpr int m3 { max(5, 6) };  // compile error: max(5, 6) not a constant expression

    int m4 { cmax(5, 6) };           // ok: may evaluate at compile-time or runtime
    const int m5 { cmax(5, 6) };     // ok: may evaluate at compile-time or runtime
    constexpr int m6 { cmax(5, 6) }; // okay: must evaluate at compile-time

    return 0;
}
```

- Constexpr functions can also be evaluated at runtime! In which case they will return a non-constexpr result (DUH!).
  
```ad-note
Remember that function call to a normal function are not allowed in constant expressions.
```

```ad-info
To evaluate at compile-time, two other things must also be true:
- The call to the constexpr function must have arguments that are known at compile time (e.g. are constant expressions).
- All statements and expressions within the constexpr function must be evaluatable at compile-time.
```

```ad-note
Allowing functions with a constexpr return type to be evaluated at either compile-time or runtime was allowed so that a single function can serve both cases.
```

### Constexpr functions are implicitly inline
- When a constexpr function is evaluated at compile-time, the compiler must be able to see the full definition of the constexpr function prior to such function calls (so it can perform the evaluation itself). A forward declaration will not suffice in this case, even if the actual function definition appears later in the same compilation unit.
- This means that a constexpr function called in multiple files needs to have its definition included into each translation unit -- which would normally be a violation of the one-definition rule. To avoid such problems, constexpr functions are implicitly inline, which makes them exempt from the one-definition rule.
- As a result, constexpr functions are often defined in header files, so they can be \#included into any .cpp file that requires the full definition.

```ad-tip
Best practice:
Constexpr/consteval functions used in a single source file (.cpp) can be defined in the source file above where they are used.
Constexpr/consteval functions used in multiple source files should be defined in a header file so they can be included into each source file.
```