In C++, variables have different types of storage duration, which determine the lifetime and visibility of a variable in a program. The storage duration dictates how long a variable remains in memory and when it is created and destroyed. Here's an overview of the different types of storage durations in C++:

### 1. **Automatic Storage Duration**
   - **Characteristics**: 
     - Variables with automatic storage duration are created when the block in which they are defined is entered and destroyed when the block is exited.
     - This is the default storage duration for local variables.
   - **Scope**: Limited to the block in which they are defined.
   - **Example**:
     ```cpp
     void foo() {
         int x = 42; // 'x' has automatic storage duration
     } // 'x' is destroyed here
     ```

### 2. **Static Storage Duration**
   - **Characteristics**:
     - Variables with static storage duration are created when the program starts and destroyed when the program ends.
     - They retain their value across multiple function calls.
     - Static variables inside a function keep their value between function calls.
   - **Scope**: Can be limited to the block, file, or have global scope.
   - **Types**:
     - **Global variables**: Defined outside all functions.
     - **Static local variables**: Defined within a function with the `static` keyword.
     - **Static member variables**: Defined in a class with the `static` keyword.
   - **Example**:
     ```cpp
     int global_var = 100; // 'global_var' has static storage duration

     void foo() {
         static int counter = 0; // 'counter' has static storage duration
         counter++;
     }
     ```

### 3. **Thread Storage Duration**
   - **Characteristics**:
     - Variables with thread storage duration are created when the thread in which they are defined starts and destroyed when the thread ends.
     - Each thread has its own instance of the variable.
   - **Scope**: Can be limited to the block, file, or have global scope, but each thread gets its own copy.
   - **Keyword**: `thread_local`
   - **Example**:
     ```cpp
     thread_local int thread_var = 0; // 'thread_var' has thread storage duration
     ```

### 4. **Dynamic Storage Duration**
   - **Characteristics**:
     - Variables with dynamic storage duration are allocated manually by the programmer using `new`, `new[]`, `malloc`, or similar functions and must be explicitly deallocated using `delete`, `delete[]`, `free`, etc.
     - The variable exists until it is explicitly deallocated.
   - **Scope**: Can be accessed through pointers as long as the memory is allocated.
   - **Example**:
     ```cpp
     int* p = new int(42); // 'p' points to a dynamically allocated int
     delete p; // Memory is freed here
     ```

### **Summary Table**

| Storage Duration     | Created When                 | Destroyed When              | Scope                                      |
|----------------------|------------------------------|-----------------------------|--------------------------------------------|
| **Automatic**        | Block entry                  | Block exit                  | Limited to the block                       |
| **Static**           | Program start (first entry)  | Program end                 | Block, file, or global                     |
| **Thread**           | Thread start                 | Thread end                  | Block, file, or global (per-thread copy)   |
| **Dynamic**          | Explicit allocation (e.g., `new`) | Explicit deallocation (e.g., `delete`) | As long as memory is allocated |

These different types of storage durations help C++ manage memory effectively, depending on the specific needs of your program.