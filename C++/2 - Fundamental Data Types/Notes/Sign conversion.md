---
tags:
  - cpp
  - note
  - fundamental
---

- Signed **integral** values can be converted to unsigned integral values, and vice-versa, using a static cast.

---

- If the value being converted can be represented in the destination type, the converted value will remain unchanged (only the type will change). For example:

```cpp
#include <iostream>

int main()
{
    unsigned int u1 { 5 };
    // Convert value of u1 to a signed int
    int s1 { static_cast<int>(u1) };
    std::cout << s1 << '\n'; // prints 5

    int s2 { 5 };
    // Convert value of s2 to an unsigned int
    unsigned int u2 { static_cast<unsigned int>(s2) };
    std::cout << u2 << '\n'; // prints 5

    return 0;
}
```

- This prints:
```ad-Answer
5
5
```

---

- If the value being converted cannot be represented in the destination type:
	- If the destination type is [[unsigned integer|unsigned]], the value will be [[Modulo Wrap|modulo wrapped]].
	- If the destination type is [[signed integers|signed]], the value is [[Implementation-defined|implementation-defined]] prior to C++20, and will be modulo wrapped as of C++20.
- Here’s an example of converting two values that are not representable in the destination type (assuming 32-bit integers):

```cpp
#include <iostream>

int main()
{
    int s { -1 };
    std::cout << static_cast<unsigned int>(s) << '\n'; // prints 4294967295

    unsigned int u { 4294967295 }; // largest 32-bit unsigned int
    std::cout << static_cast<int>(u) << '\n'; // implementation-defined prior to C++20, -1 as of C++20

    return 0;
}
```

- As of C++20, this produces the result:

```ad-Answer
4294967295
-1
```

- Signed int value `-1` cannot be represented as an unsigned int. The result modulo wraps to unsigned int value `4294967295`.
- Unsigned int value `4294967295` cannot be represented as a signed int. Prior to C++20, the result is implementation defined (but will probably be `-1`). As of C++20, the result will modulo wrap to `-1`.