---
tags: cpp, string, literal, fundamental
---

- We can create string literals with type `std::string` by using a `s` suffix after the double-quoted string literal. The `s` must be lower case.

```cpp
#include <iostream>
#include <string> // for std::string

int main()
{
    using namespace std::string_literals; // easy access to the s suffix

    std::cout << "foo\n";   // no suffix is a C-style string literal
    std::cout << "goo\n"s;  // s suffix is a std::string literal

    return 0;
}
```

- The “s” suffix lives in the namespace `std::literals::string_literals`. We recommend `using namespace std::string_literals`, which imports only the literals for `std::string`.