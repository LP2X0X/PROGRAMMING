### Inline expansion
- **Inline expansion** is a process **by the compiler** where a function call is replaced by the code from the called function’s definition. This allows us to avoid the overhead of those calls, while preserving the results of the code.
- [[Performance overhead of function called|Beyond removing the cost of function call]], inline expansion can also allow the compiler to optimize the resulting code more efficiently -- for example, because the expression `((5 < 6) ? 5 : 6)` is now a constant expression, the compiler could further optimize the first statement in `main()` to `std::cout << 5 << '\n';`.

### The inline keyword, modernly
- In previous chapters, we mentioned that you should not implement functions (with external linkage) in header files, because when those headers are included into multiple .cpp files, the function definition will be copied into multiple .cpp files. These files will then be compiled, and the linker will throw an error because it will note that you’ve defined the same function more than once, which is a violation of the one-definition rule.
- In modern C++, the term `inline` has evolved to mean “multiple definitions are allowed”. Thus, an inline function is one that is allowed to be defined in multiple translation units (without violating the ODR).
- Inline functions have two primary requirements:
	- The compiler needs to be able to see the full definition of an inline function in each translation unit where the function is used (a forward declaration will not suffice on its own). The definition can occur after the point of use if a forward declaration is also provided. Only one such definition can occur per translation unit, otherwise a compilation error will occur.
	- Every definition for an inline function must be identical, otherwise undefined behavior will result.
```ad-note
The linker will consolidate all inline function definitions for an identifier into a single definition (thus still meeting the requirements of the one definition rule).
```

```ad-tip
Inline functions are typically defined in header files, where they can be \#included into the top of any code file that needs to see the full definition of the identifier.
```

---

In C++, the ability to define `inline` functions, variables, or objects multiple times across different translation units is primarily aimed at improving performance and maintaining code organization. Here's a detailed explanation of the purpose and benefits of this feature:

### **1. Avoiding Multiple Definition Errors (One Definition Rule)**
In traditional C++ (before the introduction of the `inline` keyword for variables), if you defined a function or a variable in a header file and included that header file in multiple `.cpp` files, it would lead to multiple definition errors during the linking stage. This is because the linker would find multiple instances of the same function or variable in different translation units, violating the One Definition Rule (ODR).

**Solution with `inline`**:
- When you declare a function, variable, or object as `inline`, the compiler treats each definition as the same entity across all translation units. This means that the linker does not throw multiple definition errors, even if the `inline` entity appears in multiple translation units.
- This allows you to define functions, constants, or objects in header files without worrying about ODR violations.

### **2. Improving Performance**
The primary purpose of `inline` functions is to suggest to the compiler that it should replace a function call with the function's body wherever the function is invoked, thus avoiding the overhead of a function call. This can lead to performance improvements in cases where the function is small and called frequently.

**Benefits**:
- **Reduced Function Call Overhead**: By replacing the function call with the actual function body, the overhead of pushing and popping the stack, passing arguments, and jumping to the function's code is eliminated.
- **Potential for Further Optimization**: Inlining a function allows the compiler to optimize the resulting code better, potentially eliminating unnecessary variables or simplifying expressions.

### **3. Consistency and Code Organization**
With `inline` variables (introduced in C++17), you can define variables in a header file and include them across multiple translation units without causing linker errors. This is particularly useful for constants or configurations that are used across different parts of a program.

**Advantages**:
- **Header-Only Libraries**: You can create header-only libraries where everything, including functions and variables, is defined in headers. This makes the library easier to distribute and use, as there's no need to link against separate binary files.
- **Consistency**: By defining `inline` functions and variables in headers, you ensure that every translation unit sees the same definition, leading to consistent behavior across your program.

### **4. Example: Inline Functions**
```cpp
// math_utils.h
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

inline int add(int a, int b) {
    return a + b;
}

#endif // MATH_UTILS_H
```

```cpp
// file1.cpp
#include "math_utils.h"
int main() {
    int sum = add(3, 4); // No function call overhead
    return 0;
}
```

```cpp
// file2.cpp
#include "math_utils.h"
int calculate() {
    int result = add(5, 7); // No function call overhead
    return result;
}
```

- **Result**: The `add` function is defined in the header file, included in multiple `.cpp` files, and will be treated as the same function across these files without causing multiple definition errors.

### **5. Example: Inline Variables**
```cpp
// config.h
#ifndef CONFIG_H
#define CONFIG_H

inline constexpr int MaxValue = 100; // Inline variable

#endif // CONFIG_H
```

```cpp
// file1.cpp
#include "config.h"
#include <iostream>

void printMaxValue() {
    std::cout << "MaxValue: " << MaxValue << std::endl;
}
```

```cpp
// file2.cpp
#include "config.h"

void checkValue(int value) {
    if (value > MaxValue) {
        std::cout << "Value exceeds MaxValue" << std::endl;
    }
}
```

- **Result**: `MaxValue` is defined in a header file and can be used across multiple translation units without causing linker errors.

### **Summary**
The purpose of allowing `inline` functions, variables, or objects to be defined multiple times in C++ is to:
- Avoid multiple definition errors while maintaining the One Definition Rule (ODR).
- Improve performance by reducing function call overhead.
- Enable better code organization and consistency by allowing definitions in headers that can be included in multiple translation units.
- Facilitate the creation of header-only libraries, which simplifies code distribution and usage.

This feature enhances the flexibility and efficiency of C++ code, particularly in large projects with multiple translation units.