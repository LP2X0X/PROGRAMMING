---
tags: cpp, term, fundamental
---

- We can override [[float#Outputting|the default precision of float]] by using an `output manipulator` function named `std::setprecision()`.
- **Output manipulators** alter how data is output, and are defined in the _iomanip_ header.

---

### Example
```cpp
#include <iomanip> // for output manipulator std::setprecision()
#include <iostream>

int main()
{
    std::cout << std::setprecision(17); // show 17 digits of precision
    std::cout << 3.33333333333333333333333333333333333333f <<'\n'; // f suffix means float
    std::cout << 3.33333333333333333333333333333333333333 << '\n'; // no suffix means double

    return 0;
}
```

- Outputs:
```ad-answer
3.3333332538604736
3.3333333333333335
```

- Because we set the precision to 17 digits using `std::setprecision()`, each of the above numbers is printed with 17 digits. But, as you can see, the numbers certainly aren’t precise to 17 digits! **And because floats are less precise than doubles, the float has more error**.

```ad-tip
Output manipulators (and input manipulators) are sticky -- meaning if you set them, they will remain set.
The one exception is `std::setw`. Some IO operations reset `std::setw`, so `std::setw` should be used every time it is needed.
```
