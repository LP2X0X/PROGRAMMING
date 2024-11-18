- **Type deduction** (also sometimes called **type inference**) is a feature that allows the compiler to deduce the type of an object from the object’s initializer. When defining a variable, type deduction can be invoked by using the `auto` keyword can be used in place of the variable’s type:

```cpp
int main()
{
    auto d { 5.0 }; // 5.0 is a double literal, so d will be deduced as a double
    auto i { 1 + 2 }; // 1 + 2 evaluates to an int, so i will be deduced as an int
    auto x { i }; // i is an int, so x will be deduced as an int

    return 0;
}
```

- Because function calls are valid expressions, we can even use type deduction when our initializer is a non-void function call:

```cpp
int add(int x, int y)
{
    return x + y;
}

int main()
{
    auto sum { add(5, 6) }; // add() returns an int, so sum's type will be deduced as an int

    return 0;
}
```

### Type deduction drops `const` from the deduced type

In most cases, type deduction will drop the `const` from deduced types. For example:

```cpp
int main()
{
    const int a { 5 }; // a has type const int
    auto b { a };      // b has type int (const dropped)

    return 0;
}
```

In the above example, `a` has type `const int`, but when deducing a type for variable `b` using `a` as the initializer, type deduction deduces the type as `int`, not `const int`.

If you want a deduced type to be const, you must supply the `const` yourself as part of the definition:

```cpp
int main()
{
    const int a { 5 };  // a has type const int
    const auto b { a }; // b has type const int (const dropped but reapplied)


    return 0;
}
```

In this example, the type deduced from `a` will be `int` (the `const` is dropped), but because we’ve re-added a `const` qualifier during the definition of variable `b`, variable `b` will have type `const int`.

### Type deduction for string literals

For historical reasons, string literals in C++ have a strange type. Therefore, the following probably won’t work as expected:

```cpp
auto s { "Hello, world" }; // s will be type const char*, not std::string
```

If you want the type deduced from a string literal to be `std::string` or `std::string_view`, you’ll need to use the `s` or `sv` literal suffixes: 

```cpp
#include <string>
#include <string_view>

int main()
{
    using namespace std::literals; // easiest way to access the s and sv suffixes

    auto s1 { "goo"s };  // "goo"s is a std::string literal, so s1 will be deduced as a std::string
    auto s2 { "moo"sv }; // "moo"sv is a std::string_view literal, so s2 will be deduced as a std::string_view

    return 0;
}
```

But in such cases, it may be better to not use type deduction.

```ad-tip
Best practice:
Use type deduction for your variables when the type of the object doesn’t matter.
Favor an explicit type when you require a specific type that differs from the type of the initializer, or when your object is used in a context where making the type obvious is useful.
```