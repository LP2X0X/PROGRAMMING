- In some applications, certain symbolic constants may need to be used throughout your code (not just in one location). These can include physics or mathematical constants that don’t change (e.g. pi or Avogadro’s number), or application-specific “tuning” values (e.g. friction or gravity coefficients). Instead of redefining these constants in every file that needs them (a violation of the “Don’t Repeat Yourself” rule), it’s better to declare them once in a central location and use them wherever needed. That way, if you ever need to change them, you only need to change them in one place, and those changes can be propagated out.
- Inline variables, which are variables that can have more than one definition, so long as those definitions are identical. By making our constexpr variables inline, we can define them in a header file and then \#include them into any .cpp file that requires them. This avoids both ODR violations and the downside of duplicated variables.
- constants.h:

```cpp
#ifndef CONSTANTS_H
#define CONSTANTS_H

// define your own namespace to hold constants
namespace constants
{
    inline constexpr double pi { 3.14159 }; // note: now inline constexpr
    inline constexpr double avogadro { 6.0221413e23 };
    inline constexpr double myGravity { 9.2 }; // m/s^2 -- gravity is light on this planet
    // ... other related constants
}
#endif
```

- main.cpp:

```cpp
#include "constants.h"

#include <iostream>

int main()
{
    std::cout << "Enter a radius: ";
    double radius{};
    std::cin >> radius;

    std::cout << "The circumference is: " << 2 * radius * constants::pi << '\n';

    return 0;
}
```

- We can include `constants.h` into as many code files as we want, but these variables will only be instantiated once and shared across all code files.

- This method does retain the downside of requiring every file that includes the constants header be recompiled if any constant value is changed.

- Reference: https://www.learncpp.com/cpp-tutorial/sharing-global-constants-across-multiple-files-using-inline-variables/