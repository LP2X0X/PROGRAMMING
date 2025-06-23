---
tags: cpp, operators, fundamental
---

Arithmetic operators in C++ are used to perform basic mathematical operations like addition, subtraction, multiplication, division, and more. These operators can be used with built-in data types such as integers and floating-point numbers, as well as with user-defined types if overloaded appropriately.

### **List of Arithmetic Operators**

1. **Addition (`+`)**:
   - **Usage**: Adds two operands.
   - **Example**: 
     ```cpp
     int a = 5, b = 3;
     int result = a + b; // result is 8
     ```

2. **Subtraction (`-`)**:
   - **Usage**: Subtracts the second operand from the first.
   - **Example**:
     ```cpp
     int a = 5, b = 3;
     int result = a - b; // result is 2
     ```

3. **Multiplication (`*`)**:
   - **Usage**: Multiplies two operands.
   - **Example**:
     ```cpp
     int a = 5, b = 3;
     int result = a * b; // result is 15
     ```

4. **Division (`/`)**: ^2747fe
   - **Usage**: Divides the first operand by the second.
   - **Note**: When both operands are integers, the result is an integer (integer division), and any fractional part is discarded.
   - **Example**:
     ```cpp
     int a = 5, b = 2;
     int result = a / b; // result is 2 (integer division)

     double c = 5.0, d = 2.0;
     double result2 = c / d; // result2 is 2.5 (floating-point division)
     ```

1. **Modulus (`%`)**:
   - **Usage**: Returns the remainder of the division of the first operand by the second. It is only valid for integer operands.
	```ad-note
	The remainder operator can also work with negative operands. `x % y` always returns results with the sign of _x_.
	```
   - **Example**:
    ```cpp
    int a = 5, b = 3;
    int result = a % b; // result is 2
    ```

6. **Unary Minus (`-`)**:
   - **Usage**: Negates the value of the operand.
   - **Example**:
     ```cpp
     int a = 5;
     int result = -a; // result is -5
     ```

7. **Unary Plus (`+`)**:
   - **Usage**: Returns the value of the operand as-is. It is mostly redundant but can be used for clarity in expressions.
   - **Example**:
     ```cpp
     int a = 5;
     int result = +a; // result is 5
     ```

### **Operator Precedence and Associativity**
- **Precedence**: Determines the order in which operators are evaluated in expressions.
  - For example, `*`, `/`, and `%` have higher precedence than `+` and `-`.
- **Associativity**: Determines the direction in which operators of the same precedence level are evaluated.
  - Most arithmetic operators are left-associative, meaning they are evaluated from left to right.

### **Examples of Combined Usage**

```cpp
#include <iostream>

int main() {
    int a = 10;
    int b = 3;
    
    // Addition, subtraction, multiplication
    int sum = a + b;      // sum is 13
    int diff = a - b;     // diff is 7
    int prod = a * b;     // prod is 30
    
    // Division and modulus
    int quotient = a / b; // quotient is 3 (integer division)
    int remainder = a % b; // remainder is 1
    
    // Combined expressions
    int result = (a + b) * (a - b) / b; // result is 21

    // Output results
    std::cout << "sum: " << sum << std::endl;
    std::cout << "diff: " << diff << std::endl;
    std::cout << "prod: " << prod << std::endl;
    std::cout << "quotient: " << quotient << std::endl;
    std::cout << "remainder: " << remainder << std::endl;
    std::cout << "result: " << result << std::endl;

    return 0;
}
```

### **Output**:
```
sum: 13
diff: 7
prod: 30
quotient: 3
remainder: 1
result: 21
```

### **Summary**
- **Addition (`+`)** and **Subtraction (`-`)**: Basic arithmetic operations for adding and subtracting numbers.
- **Multiplication (`*`)** and **Division (`/`)**: Used for multiplying and dividing numbers, with division having special rules for integers.
- **Modulus (`%`)**: Returns the remainder from division, applicable only to integers.
- **Unary Plus (`+`)** and **Unary Minus (`-`)**: Used to return the value as-is or negate it.

Understanding how these operators work, especially in combination, is fundamental to programming in C++.