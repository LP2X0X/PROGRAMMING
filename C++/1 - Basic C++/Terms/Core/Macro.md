---
tags: cpp, term, fundamental
---
 A **macro** is a label defined in the source code that is replaced by its value by the [[Preprocessor|preprocessor]] before compilation.
- There are two basic types of macros: _object-like macros_, and _function-like macros_.
	- _Function-like macros_ act like functions, and serve a similar purpose. Their use is generally considered unsafe, and almost anything they can do can be done by a normal function.
	- _Object-like macros_ can be defined in one of two ways:

```cpp
#define IDENTIFIER // Most further occurrences of the identifier is removed and replaced by nothing!
#define IDENTIFIER substitution_text
// The top definition has no substitution text, whereas the bottom one does. Because these are preprocessor directives (not statements), note that neither form ends with a semicolon.
```

```ad-tip
By convention, macro names are typically all uppercase, separated by underscores.
```

```ad-note
In most cases, macro substitution does not occur when a macro identifier is used within another preprocessor command, except for `#if` and `#elif` mostly.
```

---

### Examples

```cpp
#include <iostream>

#define PI 3.14159
#define CIRCLE_AREA(radius) (PI * (radius) * (radius))

int main() {
    double r = 5.0;
    std::cout << "Area of the circle: " << CIRCLE_AREA(r) << std::endl;
    return 0;
}
```

```cpp
#include <iostream>

#define TRUE 1
#define FALSE 0

int main() {
        #if TRUE
            std::cout << "This should print." << std::endl; // This will be printed
        #endif
        #if FALSE
            std::cout << "This should not (?) print." << std::endl; // This will not
        #endif
        return 0;
}
```

#### Explanation:
1. **`#define PI 3.14159`**: This defines a macro named `PI` which will be replaced by `3.14159` wherever `PI` appears in the code.

2. **`#define CIRCLE_AREA(radius) (PI * (radius) * (radius))`**: This defines a macro that takes a parameter `radius`. When `CIRCLE_AREA(r)` is used, it is replaced by the expression `(PI * (r) * (r))`.

---

### Key Points:
- **No type checking:** Macros do not perform type checking. They simply replace text during the preprocessing step.
- **Can cause issues:** If not used carefully, macros can lead to hard-to-debug code due to unexpected behavior, especially with expressions. For example, using `#define SQUARE(x) x*x` can give unexpected results if used like `SQUARE(1+2)`, because it will expand to `1+2*1+2`.
- **Better alternatives:** In many cases, inline functions or `const` variables are preferable to macros in C++ because they offer type safety and better debugging.

---

### Macro vs. Inline Functions:
- **Macros** are expanded by the preprocessor, while **inline functions** are expanded by the compiler. 
- **Inline functions** have the advantage of type checking, which macros do not provide.

---

### Predefined Macros
- C++ has a number of predefined macros, including the following:
	- `__DATE__`: Current date formatted as `MMM DD YYYY`.
	- `__TIME__`: Current time formatted as `HH:MM:SS`.
	- `__FILE__`: Current filename.
	- `__LINE__`: Current line number.

---

### Undefining a Macro
- Once defined, a macro can be undefined with the `#undef` command. Using the macro after that point will result in a compile error.

```cpp
#include <iostream>

using namespace std;

#define TEST 1

int main() {

	  #ifdef TEST
	
	    cout << "TEST defined\n";
	
	  #else
	
	    cout << "TEST undefined\n";
	
	  #endif
	
	  #undef TEST
	
	  #ifdef TEST
	
	    cout << "TEST defined\n";
	
	  #else
	
	    cout << "TEST undefined\n";
	
	  #endif

}
```

- For complex operations or when dealing with data types, consider using `const`, `constexpr`, or inline functions instead of macros to avoid potential pitfalls.