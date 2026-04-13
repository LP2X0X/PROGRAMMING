**Conditional compilation** in C++ is a feature provided by the [[Preprocessor|preprocessor]] that **allows you to compile certain parts of your code based on specific conditions**. This is particularly useful for platform-specific code, debugging, or including/excluding features based on defined conditions.

---

### Basic Syntax

- Conditional compilation is achieved using [[Preprocessor Directive|preprocessor directives]] like `#if`, `#ifdef`, `#ifndef`, `#else`, `#elif`, and `#endif`. Here's how they work:
	- **`#if`**: Tests a compile-time condition.
	- **`#ifdef`**: Checks if a macro is defined.
	- **`#ifndef`**: Checks if a macro is not defined.
	- **`#else`**: Provides an alternative block if the previous condition is false.
	- **`#elif`**: Else-if, combines `#else` and `#if`.
	- **`#endif`**: Ends the conditional block.

--- 

### Example 1: Platform-Specific Code

```cpp
#include <iostream>

#define WINDOWS

int main() {
#ifdef WINDOWS
    std::cout << "This code is compiled for Windows." << std::endl;
#elif defined(LINUX)
    std::cout << "This code is compiled for Linux." << std::endl;
#else
    std::cout << "This code is compiled for an unknown platform." << std::endl;
#endif
    return 0;
}
```

### Explanation:
- **`#ifdef WINDOWS`**: If `WINDOWS` is defined, the code inside this block will be compiled.
- **`#elif defined(LINUX)`**: If `WINDOWS` is not defined but `LINUX` is, then this block will be compiled.
- **`#else`**: If neither `WINDOWS` nor `LINUX` is defined, this block will be compiled.
- **`#endif`**: Ends the conditional compilation block.

---

### Example 2: Debugging

```cpp
#include <iostream>

// #define DEBUG  // Uncomment this line to enable debugging

int main() {
#ifdef DEBUG
    std::cout << "Debug mode is ON" << std::endl;
#endif
    std::cout << "Running the main program." << std::endl;
    return 0;
}
```

### Explanation:
- **`#ifdef DEBUG`**: If `DEBUG` is defined, the debugging message will be printed.
- **`#endif`**: Ends the conditional block.

--- 

### Example 3: Compile code based on a version number

```cpp
#define VERSION 2

#if VERSION == 1
    std::cout << "Version 1\n";
#elif VERSION == 2
    std::cout << "Version 2\n";
#else
    std::cout << "Unknown version\n";
#endif
```

### Explanation:
- **`#if VERSION == 1`**: If `VERSION` is equal 1, then print to screen Version 1. The same goes for other version.
- **`#endif`**: Ends the conditional block.

--- 

### Use Cases:
- **Platform-specific code:** To compile code differently depending on the operating system or hardware.
- **Debugging:** To include/exclude debug code.
- **Feature toggles:** To include/exclude features based on configuration.

---

### Tips:
- **Avoid excessive use:** Overuse of conditional compilation can make the code hard to read and maintain.
- **Consider alternatives:** Sometimes runtime checks or polymorphism can be used instead of conditional compilation.

```ad-warning
Conditional compilation is a powerful tool in C++, but it should be used judiciously to keep code maintainable and clear.
```