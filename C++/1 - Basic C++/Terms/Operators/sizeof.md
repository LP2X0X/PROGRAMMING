---
tags: cpp, operator, fundamental
---

The `sizeof` operator in C++ is a **compile-time** [[Arity|unary]] [[C++/1 - Basic C++/Terms/Operators/Operator|operator]] that returns the size, in bytes, of a given type or object. It is widely used to determine the memory footprint of variables, data types, or even objects. Here’s a detailed explanation of how the `sizeof` operator works and its common use cases:

### Syntax

- **For types:**  
  ```cpp
  sizeof(type);
  ```

- **For objects/variables:**  
  ```cpp
  sizeof(object);
  ```

### Key Characteristics

1. **Compile-Time Evaluation:**
   - The `sizeof` operator is evaluated at compile time, which means that the size it returns is determined during the compilation process, not at runtime.

2. **Result Type:**
   - The result of `sizeof` is of type '[[size_t|std::size_t]]', which is an unsigned integral type defined in the `<cstddef>` header.

3. **Returns Size in Bytes:**
   - The size returned by `sizeof` is in bytes. For example, `sizeof(int)` might return 4 on many platforms, indicating that an `int` takes 4 bytes of memory.

4. **Works on Any Type:**
   - You can use `sizeof` with primitive data types (like `int`, `char`, `float`), user-defined types (like classes and structs), and even complex expressions.

### Common Uses of `sizeof`

1. **Determine Size of Primitive Data Types:**
   ```cpp
   std::cout << "Size of int: " << sizeof(int) << " bytes" << std::endl;
   std::cout << "Size of char: " << sizeof(char) << " bytes" << std::endl;
   ```

2. **Determine Size of Structures or Classes:**
   - The size includes all member variables, padding, and alignment.
   ```cpp
   struct MyStruct {
       int a;
       double b;
       char c;
   };

   std::cout << "Size of MyStruct: " << sizeof(MyStruct) << " bytes" << std::endl;
   ```

3. **Size of Arrays:**
   - For arrays, `sizeof` returns the total size of the array, which is the size of a single element multiplied by the number of elements.
   ```cpp
   int arr[10];
   std::cout << "Size of arr: " << sizeof(arr) << " bytes" << std::endl;
   std::cout << "Number of elements in arr: " << sizeof(arr) / sizeof(arr[0]) << std::endl;
   ```

4. **Memory Allocation:**
   - `sizeof` is often used with memory allocation functions like `malloc` to ensure the correct amount of memory is allocated.
   ```cpp
   int* ptr = (int*)malloc(10 * sizeof(int));
   ```

5. **Determine Object Size:**
   - You can determine the size of an object of a class, which includes its data members and any potential padding due to alignment.
   ```cpp
   MyStruct obj;
   std::cout << "Size of obj: " << sizeof(obj) << " bytes" << std::endl;
   ```

6. **Alignment Considerations:**
   - The size returned by `sizeof` includes any padding added by the compiler to ensure proper alignment of data in memory.

### Special Considerations

- **`sizeof` with Dynamic Arrays:**
  - If you use `sizeof` on a pointer (including the name of a dynamically allocated array), it returns the size of the pointer itself, not the size of the array it points to.
  ```cpp
  int* arr = new int[10];
  std::cout << "Size of arr (pointer): " << sizeof(arr) << " bytes" << std::endl;
  ```

- **`sizeof` and Empty Classes:**
  - The size of an empty class in C++ is guaranteed to be at least 1 byte. This is to ensure that two distinct objects of the empty class have different addresses.

  ```cpp
  class Empty {};
  std::cout << "Size of Empty: " << sizeof(Empty) << " bytes" << std::endl;
  ```

- **`sizeof` and Inheritance:**
  - When dealing with inheritance, the size includes the base class and derived class members, along with any additional padding for alignment.

### Example Code

```cpp
#include <iostream>

struct MyStruct {
    int a;
    double b;
    char c;
};

class MyClass {
    int x;
    double y;
};

int main() {
    std::cout << "Size of int: " << sizeof(int) << " bytes" << std::endl;
    std::cout << "Size of MyStruct: " << sizeof(MyStruct) << " bytes" << std::endl;
    std::cout << "Size of MyClass: " << sizeof(MyClass) << " bytes" << std::endl;

    int arr[10];
    std::cout << "Size of arr: " << sizeof(arr) << " bytes" << std::endl;
    std::cout << "Number of elements in arr: " << sizeof(arr) / sizeof(arr[0]) << std::endl;

    return 0;
}
```

### Output Example

```
Size of int: 4 bytes
Size of MyStruct: 16 bytes
Size of MyClass: 16 bytes
Size of arr: 40 bytes
Number of elements in arr: 10
```

In this example, `sizeof` is used to determine the size of different types, objects, and arrays, illustrating how it can be employed in various contexts.