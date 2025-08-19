---
tags:
  - cpp
  - term
  - fundamental
---

- **Scientific notation** is a useful shorthand for writing lengthy numbers in a concise manner.
- Because it can be hard to type or display exponents in C++, we use the letter ‘e’ (or sometimes ‘E’) to represent the “times 10 to the power of” part of the equation. For example, `1.2 x 10⁴` would be written as `1.2e4`, and `5.9722 x 10²⁴` would be written as `5.9722e24`.

### Using `std::scientific` with `std::cout`:

The `std::scientific` manipulator forces floating-point numbers to be displayed in scientific notation.

```cpp
#include <iostream>
#include <iomanip> // For std::scientific and std::setprecision

int main() {
    double num = 12345.6789;

    std::cout << "Default: " << num << std::endl;

    // Scientific notation
    std::cout << "Scientific notation: " << std::scientific << num << std::endl;

    // You can also set the precision
    std::cout << "Scientific notation with precision: " << std::scientific << std::setprecision(3) << num << std::endl;

    return 0;
}
```

### Output:

```
Default: 12345.7
Scientific notation: 1.234568e+04
Scientific notation with precision: 1.235e+04
```

### Explanation:
- **Default**: The default output is in normal floating-point notation.
- **Scientific notation**: When `std::scientific` is used, the number is displayed in the form `m.nnnnnnE+e`, where `m` is the mantissa and `e` is the exponent.
- **Precision**: You can control the number of digits after the decimal point using `std::setprecision`.

### Using `printf`:

The `printf` function allows you to specify the format directly using format specifiers.

```cpp
#include <cstdio>

int main() {
    double num = 12345.6789;

    printf("Default: %f\n", num);

    // Scientific notation
    printf("Scientific notation: %e\n", num);

    // You can also set the precision
    printf("Scientific notation with precision: %.3e\n", num);

    return 0;
}
```

### Output:

```
Default: 12345.678900
Scientific notation: 1.234568e+04
Scientific notation with precision: 1.235e+04
```

### Explanation:
- **`%f`**: Prints the number in default floating-point notation.
- **`%e`**: Prints the number in scientific notation.
- **`%.3e`**: Prints the number in scientific notation with three decimal places.

### Additional Manipulators:
- **`std::fixed`**: Forces the output to be in fixed-point notation.
- **`std::setprecision`**: Sets the number of digits to display after the decimal point.

### Example:

```cpp
#include <iostream>
#include <iomanip>

int main() {
    double num = 0.0000123456789;

    std::cout << "Fixed: " << std::fixed << std::setprecision(10) << num << std::endl;
    std::cout << "Scientific: " << std::scientific << std::setprecision(10) << num << std::endl;

    return 0;
}
```

### Output:

```
Fixed: 0.0000123457
Scientific: 1.2345678900e-05
```

This example demonstrates how to switch between fixed-point and scientific notation, showing the difference in how the number is represented.