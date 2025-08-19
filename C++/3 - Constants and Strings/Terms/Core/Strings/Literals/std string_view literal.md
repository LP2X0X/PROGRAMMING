---
tags:
  - cpp
  - string
  - literal
  - fundamental
---

- We can create string literals with type `std::string_view` by using a `sv` suffix after the double-quoted string literal.
- The `std::string_view` literal is useful because it provides a way to create a non-owning, efficient reference to a string literal or any other character sequence without the overhead of copying the string data into a `std::string` object. This is particularly advantageous in scenarios where performance and memory efficiency are critical.

```cpp
#include <iostream>
#include <string>      // for std::string
#include <string_view> // for std::string_view

int main()
{
    using namespace std::string_literals;      // access the s suffix
    using namespace std::string_view_literals; // access the sv suffix

    std::cout << "foo\n";   // no suffix is a C-style string literal
    std::cout << "goo\n"s;  // s suffix is a std::string literal
    std::cout << "moo\n"sv; // sv suffix is a std::string_view literal

    return 0;
}
```