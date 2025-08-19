---
tags:
  - cpp
  - special
  - value
  - fundamental
---

- In C++, **NaN (Not a Number)** represents undefined or unrepresentable floating-point values (like the result of `0.0 / 0.0`).
- There isn't just one NaN—there are **many different kinds**, all defined by the **IEEE 754 standard**, which C++ floating-point types (like `float`, `double`) typically follow.

---

## Example 

```cpp
#include <iostream>

int main()
{
    double zero { 0.0 };

    double posinf { 5.0 / zero }; // positive infinity
    std::cout << posinf << '\n';

    double neginf { -5.0 / zero }; // negative infinity
    std::cout << neginf << '\n';

    double z1 { 0.0 / posinf }; // positive zero
    std::cout << z1 << '\n';

    double z2 { -0.0 / posinf }; // negative zero
    std::cout << z2 << '\n';

    double nan { zero / zero }; // not a number (mathematically invalid)
    std::cout << nan << '\n';

    return 0;
}
```

---

## 🔹 Categories of NaN

### 1. **Quiet NaN (qNaN)**

- Used to **propagate errors silently**.
- Most operations that involve a `qNaN` will return another `qNaN`.
- Common in compilers and standard libraries.

### 2. **Signaling NaN (sNaN)**

- Used to **signal an invalid operation** when used in computations.
- Triggers **hardware exceptions** or signals when processed (if enabled).
- Less commonly used; some systems treat them as quiet NaNs.

---

## 🔹 How NaNs Differ Internally

In IEEE 754:

- A NaN is represented by:
    - **Exponent bits all 1s**
    - **Non-zero fraction/mantissa bits**
- The **payload** (mantissa bits) can vary, making **many distinct NaN values**.

So in C++, two NaNs can have:

- Different bit patterns
    
- Different payloads (data encoded in mantissa)
    
- Different types (qNaN vs sNaN)
    

But **all NaNs compare unequal**, even to themselves.

---

## 🔹 Creating and Detecting NaNs in C++

### Creation

```cpp
#include <cmath>
#include <limits>

double nan1 = std::numeric_limits<double>::quiet_NaN();
double nan2 = std::nan("1");   // Using payload
```

### Detection

```cpp
#include <cmath>

if (std::isnan(value)) {
    // value is NaN
}
```

---

## 🔹 Summary Table

|Kind|Triggers Exception?|Carries Payload?|Use Case|
|---|---|---|---|
|**qNaN**|No|Yes|Silent error propagation|
|**sNaN**|Yes (if supported)|Yes|Alerting on invalid use|
