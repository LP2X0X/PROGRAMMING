## The remainder operator (`operator%`)
- The **remainder operator** (also commonly called the **modulo operator** or **modulus operator**) is an operator that returns the remainder after doing an integer division.
```ad-note
The remainder operator only works with integer operands.
```
- The remainder operator can also work with negative operands. `x % y` always returns results with the sign of _x_.

## Exponent operator?
- C++ does not include an exponent operator. To do exponents in C++, \#include the \<cmath\> header, and use the pow() function.
```ad-note
Note that the parameters (and return value) of function pow() are of type double. Due to rounding errors in floating point numbers, the results of pow() may not be precise (even if you pass it integers or whole numbers).
```