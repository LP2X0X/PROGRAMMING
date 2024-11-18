`NDEBUG` is a macro in C and C++ used to control debugging-related code. It stands for "No Debug" and is typically used with the `assert` macro to disable debugging checks in production code.

### How `NDEBUG` Works:

1. **Assertion Control:**
   In C and C++, the `assert()` macro is commonly used to insert diagnostic checks in the code, typically for debugging. If the condition inside `assert()` evaluates to false, the program will terminate with an error message showing the file name and line number where the assertion failed.

   Example without `NDEBUG`:
   ```c
   #include <assert.h>
   
   int main() {
       int x = 0;
       assert(x != 0); // Program will terminate here since x == 0
       return 0;
   }
   ```

2. **Disabling Assertions with `NDEBUG`:**
   When `NDEBUG` is defined, all `assert()` calls are ignored by the preprocessor, effectively disabling any assertions in the code. This is useful for production builds where performance is a priority, and you don't need debugging checks.

   Example with `NDEBUG`:
   ```c
   #define NDEBUG
   #include <assert.h>

   int main() {
       int x = 0;
       assert(x != 0); // This will be ignored because NDEBUG is defined
       return 0;
   }
   ```

3. **Usage in Build Configurations:**
   - In **debug** builds, assertions are enabled by default because `NDEBUG` is not defined.
   - In **release** or **production** builds, `NDEBUG` is typically defined to disable all assertions, optimizing the performance of the final executable.

### How to Define `NDEBUG`:
You can define `NDEBUG` manually in your source code or pass it as a compiler flag.

- **Defining in Code:**
  ```c
  #define NDEBUG
  ```

- **Defining via Compiler Flag:**
  ```bash
  gcc -DNDEBUG -o myprogram myprogram.c
  ```

### Conclusion:
`NDEBUG` is used to control whether assertions are active in your code. By defining `NDEBUG`, you disable assertions, which can improve performance in production environments but also means you lose the safety checks provided by `assert()`.