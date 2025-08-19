---
tags:
  - cpp
  - term
  - fundamental
---

In C++, a "magic number" is a hard-coded value that appears directly in the code (which is a [[Literals|literal]]) without explanation, making the code less readable and maintainable. These values are often arbitrary constants, and their purpose isn't immediately clear to someone reading the code.

### Example of Magic Numbers
```cpp
int area = length * 5;
```

In this example, the number `5` is a magic number. It's unclear why `5` is used, and it might confuse someone maintaining the code later.

### How to Avoid Magic Numbers

1. **Use Constants**: Define meaningful constant names for your magic numbers.
    ```cpp
    const int WIDTH = 5;
    int area = length * WIDTH;
    ```

2. **Use Enumerations**: When dealing with a set of related constants, you can use an enum.
    ```cpp
    enum Shape { SQUARE = 4, TRIANGLE = 3, CIRCLE = 1 };
    int area = length * SQUARE;
    ```

3. **Use `constexpr`**: For compile-time constants, prefer `constexpr` over `const`.
    ```cpp
    constexpr int WIDTH = 5;
    int area = length * WIDTH;
    ```

### Why Avoid Magic Numbers?
- **Readability**: Code becomes easier to understand.
- **Maintainability**: If you need to change the value, you only need to update the constant, not every instance of the number.
- **Avoid Errors**: Magic numbers can lead to errors if used inconsistently.

By replacing magic numbers with named constants or enums, you make your code clearer, more maintainable, and less error-prone.