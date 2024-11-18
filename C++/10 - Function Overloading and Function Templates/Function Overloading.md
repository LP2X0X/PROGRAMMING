**Function Overloading** in C++ is a feature that allows you to create multiple functions with the same name but different parameters. It enables a more intuitive use of functions, as you can have different versions of a function that perform similar but slightly different tasks based on the number or types of arguments provided.

### Key Concepts of Function Overloading

1. **Same Function Name**: All overloaded functions have the same name.
2. **Different Parameter Lists**:
   - **Number of Parameters**: Functions can be differentiated by the number of parameters.
   - **Type of Parameters**: Functions can be differentiated by the types of parameters passed.
   - **Order of Parameters**: Changing the order of different types can also create a unique function signature.
3. **Return Type**: The return type alone is not sufficient for function overloading. The compiler requires the function's signature (name + parameter list) to be different.

### Example of Function Overloading in C++

Here's an example to demonstrate how function overloading works:

```cpp
#include <iostream>

// Function to display an integer
void display(int num) {
    std::cout << "Integer: " << num << std::endl;
}

// Function to display a double
void display(double num) {
    std::cout << "Double: " << num << std::endl;
}

// Function to display a string
void display(const std::string& str) {
    std::cout << "String: " << str << std::endl;
}

int main() {
    display(5);           // Calls the version for int
    display(3.14);        // Calls the version for double
    display("Hello");     // Calls the version for string
    
    return 0;
}
```

### Rules for Function Overloading

1. **Parameter List**: Functions must differ in their parameter list. You cannot overload functions that have the exact same parameter types and count.
2. **Return Type Ignored**: The compiler does not consider the return type of the function for distinguishing between overloaded functions.
3. **Default Arguments**: Overloading functions with default arguments should be done carefully to avoid ambiguity.

### Ambiguity in Function Overloading

Overloading can sometimes lead to ambiguity if the compiler cannot decide which function to call. This usually occurs when the parameter list matches more than one overloaded function or when implicit type conversions create confusion.

**Example of Ambiguous Overloading**:

```cpp
void display(int num);
void display(double num);

display(5);      // Calls display(int)
display(5.0);    // Calls display(double)

display('A');    // Ambiguous, as 'A' can be promoted to either int or double
```

### Benefits of Function Overloading

- **Code Clarity**: Makes the code more readable and maintainable.
- **Reusability**: Allows you to use the same function name for different types of operations, enhancing code reusability.

### Common Use Cases

- **Mathematical Operations**: Overloading functions like `add`, `multiply`, etc., for different data types (e.g., `int`, `float`, `double`).
- **I/O Operations**: Displaying or formatting output for different types of data.
- **Constructors**: Class constructors can be overloaded to allow initializing objects in various ways.

Function overloading is a fundamental aspect of **polymorphism** in C++ that enhances the flexibility and readability of your code.