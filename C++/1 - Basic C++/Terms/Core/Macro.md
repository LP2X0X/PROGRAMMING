In C++, a macro is a fragment of code that is given a name. Whenever the name is used, it is replaced by the contents of the macro. Macros are handled by the preprocessor, which is a step that occurs before the actual compilation of the code. 

Macros in C++ are defined using the `#define` directive. Here is an example:

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

### Explanation:
1. **`#define PI 3.14159`**: This defines a macro named `PI` which will be replaced by `3.14159` wherever `PI` appears in the code.

2. **`#define CIRCLE_AREA(radius) (PI * (radius) * (radius))`**: This defines a macro that takes a parameter `radius`. When `CIRCLE_AREA(r)` is used, it is replaced by the expression `(PI * (r) * (r))`.

### Key Points:
- **No type checking:** Macros do not perform type checking. They simply replace text during the preprocessing step.
- **Can cause issues:** If not used carefully, macros can lead to hard-to-debug code due to unexpected behavior, especially with expressions. For example, using `#define SQUARE(x) x*x` can give unexpected results if used like `SQUARE(1+2)`, because it will expand to `1+2*1+2`.
- **Better alternatives:** In many cases, inline functions or `const` variables are preferable to macros in C++ because they offer type safety and better debugging.

### Macro vs. Inline Functions:
- **Macros** are expanded by the preprocessor, while **inline functions** are expanded by the compiler. 
- **Inline functions** have the advantage of type checking, which macros do not provide.

### Predefined Macros
- C++ has a number of predefined macros, including the following:
	- `__DATE__`: Current date formatted as `MMM DD YYYY`.
	- `__TIME__`: Current time formatted as `HH:MM:SS`.
	- `__FILE__`: Current filename.
	- `__LINE__`: Current line number.

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

For complex operations or when dealing with data types, consider using `const`, `constexpr`, or inline functions instead of macros to avoid potential pitfalls.