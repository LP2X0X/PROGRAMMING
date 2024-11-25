- Enumerators have values that are of an integral type. But what integral type? The specific integral type used to represent the value of enumerators is called the enumeration’s **underlying type** (or **base**).
- For unscoped enumerations, the C++ standard does not specify which specific integral type should be used as the underlying type, so the choice is implementation-defined. Most compilers will use `int` as the underlying type (meaning an unscoped enum will be the same size as an `int`), unless a larger type is required to store the enumerator values. But you shouldn’t assume this will hold true for every compiler or platform.
- It is possible to explicitly specify an underlying type for an enumeration. The underlying type must be an integral type. For example, if you are working in some bandwidth-sensitive context (e.g. sending data over a network) you may want to specify a smaller type for your enumeration:

```cpp
#include <cstdint>  // for std::int8_t
#include <iostream>

// Use an 8-bit integer as the enum underlying type
enum Color : std::int8_t
{
    black,
    red,
    blue,
};

int main()
{
    Color c{ black };
    std::cout << sizeof(c) << '\n'; // prints 1 (byte)

    return 0;
}
```

```ad-tip
**Best practice:** Specify the base type of an enumeration only when necessary.
```
