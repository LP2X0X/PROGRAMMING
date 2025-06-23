---
tags: cpp, note
---

---
```ad-summary
Ensure that the expressions (or function calls) you write are not dependent on operand (or argument) evaluation order.
```
---

- In most cases, the order of evaluation for operands and function arguments is unspecified, meaning they may be evaluated in any order.
```ad-note
The Clang compiler evaluates arguments in left-to-right order. The GCC compiler evaluates arguments in right-to-left order.
```
- Consider the following expression:
```cpp
a * b + c * d
```

- We know from the precedence and associativity rules above that this expression will be grouped as if we had typed:
```cpp
(a * b) + (c * d) // We don't know if a * b will be calculated first of c * d
```

- If `a` is `1`, `b` is `2`, `c` is `3`, and `d` is `4`, this expression will always compute the value `14`.
- However, the precedence and associativity rules only tell us **how operators and operands are grouped and the order in which value computation will occur**. **They do not tell us the order in which the operands or subexpressions are evaluated.** The compiler is free to evaluate operands `a`, `b`, `c`, or `d` in any order. The compiler is also free to calculate `a * b` or `c * d` first.
- For most expressions, this is irrelevant. In our sample expression above, it doesn’t matter whether in which order variables `a`, `b`, `c`, or `d` are evaluated for their values: the value calculated will always be `14`. There is no ambiguity here.
- But it is possible to write expressions where the order of evaluation does matter. Consider this program, which contains a mistake often made by new C++ programmers:
```cpp
#include <iostream>

int getValue()
{
    std::cout << "Enter an integer: ";

    int x{};
    std::cin >> x;
    return x;
}

void printCalculation(int x, int y, int z)
{
    std::cout << x + (y * z);
}

int main()
{
    printCalculation(getValue(), getValue(), getValue()); // this line is ambiguous

    return 0;
}
```

- If you run this program and enter the inputs `1`, `2`, and `3`, you might assume that this program would calculate `1 + (2 * 3)` and print `7`. But that is making the assumption that the arguments to `printCalculation()` will evaluate in left-to-right order (so parameter `x` gets value `1`, `y` gets value `2`, and `z` gets value `3`). If instead, the arguments evaluate in right-to-left order (so parameter `z` gets value `1`, `y` gets value `2`, and `x` gets value `3`), then the program will print `5` instead.

```ad-note
The Clang compiler evaluates arguments in left-to-right order. The GCC compiler evaluates arguments in right-to-left order.
```