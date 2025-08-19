---
tags:
  - cpp
  - term
  - fundamental
---

- The **precision** of a floating point type defines how many [[Significant digits|significant digits]] it can represent without information loss. This means that the output value of a float type will only be accurate up to the number of significant digits.

### Example

```cpp
#include <iomanip> // for std::setprecision()
#include <iostream>

int main()
{
    float f { 123456789.0f }; // f has 10 significant digits
    std::cout << std::setprecision(9); // to show 9 digits in f
    std::cout << f << '\n';

    return 0;
}
```

```ad-answer
Output:

123456792
```

- 123456792 is greater than 123456789. The value 123456789.0 has 10 significant digits, but float values typically have 7 digits of precision (and the result of 123456792 is **precise only to 7 significant digits**).