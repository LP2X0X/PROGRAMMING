The `std::atexit` function in C++ is used to register a function that will be called automatically when `std::exit` is invoked, or when the program terminates normally (such as when `main()` returns). This provides a way to ensure that certain cleanup actions are performed at the end of a program, regardless of how it exits.

### **Syntax**

```cpp
#include <cstdlib>

int std::atexit(void (*func)()) noexcept;
```

- **`func`**: A pointer to the function you want to register. The function must have no parameters and return `void`.

- **Return Value**: `std::atexit` returns `0` if the function is successfully registered. If registration fails (which is rare), a non-zero value is returned.

### **Usage Example**

Here’s an example demonstrating how to use `std::atexit`:

```cpp
#include <iostream>
#include <cstdlib>

void cleanup1() {
    std::cout << "Cleanup function 1 called." << std::endl;
}

void cleanup2() {
    std::cout << "Cleanup function 2 called." << std::endl;
}

int main() {
    // Register cleanup functions
    std::atexit(cleanup1);
    std::atexit(cleanup2);

    std::cout << "Main function running." << std::endl;

    // Exiting the program
    std::exit(0);

    // Code here will not be executed
    std::cout << "This will never be printed." << std::endl;

    return 0;
}
```

### **Explanation of the Example**

1. **Registering Cleanup Functions**:
   - `std::atexit(cleanup1)` and `std::atexit(cleanup2)` register `cleanup1` and `cleanup2` as functions to be called when the program exits.

2. **Order of Execution**:
   - The functions are called in the reverse order of their registration. In this case, `cleanup2` will be called first, followed by `cleanup1`.

3. **`std::exit`**:
   - When `std::exit(0)` is called, the program terminates, and the registered functions are called automatically.

4. **Output**:
   - The output will be:
     ```
     Main function running.
     Cleanup function 2 called.
     Cleanup function 1 called.
     ```

   The message "This will never be printed." is not shown because `std::exit` terminates the program before reaching that line.

### **Important Points**

- **Multiple Registrations**: You can register multiple functions with `std::atexit`, and they will be called in the reverse order of their registration.
  
- **Global Static Objects**: The destructors of global and static objects are called after all `std::atexit` registered functions have been executed.

- **Error Handling**: If the registration of a function fails, `std::atexit` returns a non-zero value, though such failures are rare and typically only happen if there are too many functions registered.

- **Use Cases**: Common use cases include closing files, releasing resources, flushing buffers, or performing other cleanup tasks before the program exits.