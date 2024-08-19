In C++, numeral systems (or number systems) represent numbers in different bases, such as binary, octal, decimal, and hexadecimal. C++ provides various ways to work with these numeral systems, both for input and output.

### Common Numeral Systems in C++
1. **Binary (Base 2)**: Uses digits `0` and `1`.
2. **Octal (Base 8)**: Uses digits `0` to `7`.
3. **Decimal (Base 10)**: Uses digits `0` to `9`.
4. **Hexadecimal (Base 16)**: Uses digits `0` to `9` and letters `A` to `F` (or `a` to `f`).

### Declaring Numbers in Different Bases
C++ allows you to declare integer literals in different bases directly in your code:

- **Binary**: Prefix with `0b` or `0B`.
  ```cpp
  int binary = 0b1010;  // Binary for 10 in decimal
  ```

```ad-note
Because long literals can be hard to read, C++14 also adds the ability to use a quotation mark (‘) as a digit separator. Also note that the separator can not occur before the first digit of the value.
```
- **Octal**: Prefix with `0`.
  ```cpp
  int octal = 012;  // Octal for 10 in decimal
  ```

- **Decimal**: The default base.
  ```cpp
  int decimal = 10;  // Decimal 10
  ```

- **Hexadecimal**: Prefix with `0x` or `0X`.
  ```cpp
  int hexadecimal = 0xA;  // Hexadecimal for 10 in decimal
  ```

### Outputting Numbers in Different Bases
```ad-note
Note that once applied, the I/O manipulator remains set for future output until it is changed again.
```
You can output numbers in different bases using stream manipulators:

- **Binary**: C++ doesn't have a built-in manipulator for binary, but you can use the `<bitset>` library.
  ```cpp
  #include <iostream>
  #include <bitset>

  int main() {
      int num = 10;
      std::cout << std::bitset<8>(num) << std::endl;  // Outputs binary 00001010
      return 0;
  }
  ```

- **Octal**: Use `std::oct`.
  ```cpp
  #include <iostream>

  int main() {
      int num = 10;
      std::cout << std::oct << num << std::endl;  // Outputs 12 (octal)
      return 0;
  }
  ```

- **Decimal**: Use `std::dec`.
  ```cpp
  #include <iostream>

  int main() {
      int num = 10;
      std::cout << std::dec << num << std::endl;  // Outputs 10 (decimal)
      return 0;
  }
  ```

- **Hexadecimal**: Use `std::hex`.
  ```cpp
  #include <iostream>

  int main() {
      int num = 10;
      std::cout << std::hex << num << std::endl;  // Outputs a (hexadecimal)
      return 0;
  }
  ```

### Converting Between Numeral Systems
To convert between numeral systems, you can use `std::stoi` and `std::stol` with the appropriate base.

```cpp
#include <iostream>
#include <string>

int main() {
    std::string binary_str = "1010";
    int decimal_num = std::stoi(binary_str, nullptr, 2);  // Convert binary to decimal
    std::cout << decimal_num << std::endl;  // Outputs 10

    std::string hex_str = "A";
    decimal_num = std::stoi(hex_str, nullptr, 16);  // Convert hex to decimal
    std::cout << decimal_num << std::endl;  // Outputs 10

    return 0;
}
```

### Outputting values in binary using Format / Print Library
In C++20 and C++23, we have better options for printing binary via the new Format Library (C++20) and Print Library (C++23):
```cpp
#include <format> // C++20
#include <iostream>
#include <print> // C++23

int main()
{
    std::cout << std::format("{:b}\n", 0b1010);  // C++20, {:b} formats the argument as binary digits
    std::cout << std::format("{:#b}\n", 0b1010); // C++20, {:#b} formats the argument as 0b-prefixed binary digits

    std::println("{:b} {:#b}", 0b1010, 0b1010);  // C++23, format/print two arguments (same as above) and a newline

    return 0;
}
```

### Summary
- **Binary** literals are prefixed with `0b`.
- **Octal** literals are prefixed with `0`.
- **Decimal** is the default numeral system.
- **Hexadecimal** literals are prefixed with `0x`.
- Use stream manipulators like `std::oct`, `std::dec`, and `std::hex` for output.
- Use `std::stoi` or `std::stol` for converting strings in different bases to decimal integers. 

Understanding and working with different numeral systems in C++ is crucial when dealing with low-level programming tasks, such as bit manipulation, hardware interfacing, and memory management.