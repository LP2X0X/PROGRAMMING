---
tags: cpp, datatype, fundamental
---

- Since if you use a fixed-width integer, it may be slower than a wider type on some architectures, the fast types (std::int_fast#\_t and std::uint_fast#\_t) provide the fastest signed/unsigned integer type with a width of at least ***# bits*** (where # = 8, 16, 32, or 64).
- By fastest, we mean the integral type that can be processed most quickly by the CPU.

````ad-warning
Because the size of the fast integers is implementation-defined, your program may exhibit different behaviors on architectures where they resolve to different sizes.
```cpp
#include <cstdint>
#include <iostream>

int main()
{
    std::uint_fast16_t sometype { 0 };
    sometype = sometype - 1; // intentionally overflow to invoke wraparound behavior

    std::cout << sometype << '\n';

    return 0;
}
```
This code will produce different results depending on whether `std::uint_fast16_t` is 16, 32, or 64 bits based on the machine.
````