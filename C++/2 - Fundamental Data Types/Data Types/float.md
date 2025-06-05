---
tags: cpp, datatype, fundamental
---

- A **floating point** type variable is a variable that can hold a number with a fractional component, such as 4320.0, -3.33, or 0.01226. The _floating_ part of the name _floating point_ refers to the fact that the decimal point can “float” -- that is, it can support a variable number of digits before and after the decimal point.

---
### Types
- C++ has three fundamental floating point data types: a single-precision `float`, a double-precision `double`, and an extended-precision `long double`.

| Category       | Type        | Typical Size       |
| -------------- | ----------- | ------------------ |
| floating point | float       | 4 bytes            |
|                | double      | 8 bytes            |
|                | long double | 8, 12, or 16 bytes |

```ad-note
On modern architectures, floating-point types are conventionally implemented using one of the floating-point formats defined in the IEEE 754 standard (see [https://en.wikipedia.org/wiki/IEEE_754](https://en.wikipedia.org/wiki/IEEE_754)). As a result, `float` is almost always 4 bytes, and `double` is almost always 8 bytes.
```

---

### Definitions
```ad-important
No f suffix means double type by default.
```

```cpp
int a { 5 };      // 5 means integer
double b { 5.0 }; // 5.0 is a floating point literal (no suffix means double type by default)
float c { 5.0f }; // 5.0 is a floating point literal, f suffix means float type

int d { 0 };      // 0 is an integer
double e { 0.0 }; // 0.0 is a double
```


|Format|Range|Precision|
|---|---|---|
|IEEE 754 single-precision (4 bytes)|±1.18 x 10<sup>-38</sup> to ±3.4 x 10<sup>38</sup> and 0.0|6-9 [[Precision\|significant digits]], typically 7|
|IEEE 754 double-precision (8 bytes)|±2.23 x 10<sup>-308</sup> to ±1.80 x 10<sup>308</sup> and 0.0|15-18 significant digits, typically 16|
|x87 extended-precision (80 bits)|±3.36 x 10<sup>-4932</sup> to ±1.18 x 10<sup>4932</sup> and 0.0|18-21 significant digits|
|IEEE 754 quadruple-precision (16 bytes)|±3.36 x 10<sup>-4932</sup> to ±1.18 x 10<sup>4932</sup> and 0.0|33-36 significant digits|

- Reference: https://fabiensanglard.net/floating_point_visually_explained/index.html

---

### Outputting
- When outputting floating point numbers, `std::cout` has a default precision of 6 -- that is, it assumes all floating point variables are only significant to 6 digits (the minimum precision of a float), and hence it will truncate anything after that.
- The following program shows `std::cout` truncating to 6 digits:

```cpp
#include <iostream>

int main()
{
    std::cout << 9.87654321f << '\n';
    std::cout << 987.654321f << '\n';
    std::cout << 987654.321f << '\n';
    std::cout << 9876543.21f << '\n';
    std::cout << 0.0000987654321f << '\n';

    return 0;
}
```

```ad-answer
This program outputs:

9.87654
987.654
987654
9.87654e+006
9.87654e-005
```

```ad-note
Note that each of these only have 6 significant digits.
```

--- 

### References
- For more insight into how floating point numbers are stored in binary, check out the [float.exposed](https://float.exposed/0x3dcccccd) tool.  
- To learn more about floating point numbers and rounding errors, [floating-point-gui.de](https://floating-point-gui.de/) and [fabiensanglard.net](https://fabiensanglard.net/floating_point_visually_explained/index.html) have approachable guides on the topic.