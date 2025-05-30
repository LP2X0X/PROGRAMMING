---
tags: cpp, datatype, fundamental
---

- **Unsigned integers** are [[Integer|integers]] that can only hold non-negative whole numbers.
- To define an unsigned integer, we use the _unsigned_ keyword. By convention, this is placed before the type:
	```cpp
	unsigned short us;
	unsigned int ui;
	unsigned long ul;
	unsigned long long ull;
	```
- If an unsigned value is out of range, it is **divided by one greater than the largest number of the type, and only the remainder kept**.

---

### Why You Might Want to Avoid Unsigned Integers

1. **Unintended Bugs from Overflow**:
   - Unsigned integers can wrap around when they overflow. For example, subtracting a larger unsigned integer from a smaller one can result in a very large number instead of a negative number. This can lead to subtle bugs that are difficult to detect.
   - Example:
     ```cpp
     unsigned int a = 1;
     unsigned int b = 2;
     unsigned int c = a - b;  // c becomes 4294967295 instead of -1
     ```

2. **Unexpected Behavior in Loops**:
   - When using unsigned integers in loops, you can accidentally create infinite loops or miss terminating conditions if the loop index goes below zero, as it will wrap around to the maximum value.
   - Example:
     ```cpp
     for (unsigned int i = 10; i >= 0; --i) {
         std::cout << i << std::endl;
     }
     // This will result in an infinite loop because when i becomes 0 and then decrements,
     // it wraps around to the maximum value of unsigned int.
     ```

3. **Comparisons with Signed Integers**:
   - Comparing signed and unsigned integers can lead to unexpected results because of implicit type conversions. When you compare a signed integer with an unsigned integer, **the signed integer is often converted to an unsigned type**, which can lead to incorrect comparisons.
   - Example:
     ```cpp
     int a = -1;
     unsigned int b = 1;
     if (a < b) {
         std::cout << "This will not print as expected!" << std::endl;
     }
     // Here, -1 (signed) is converted to an unsigned integer, leading to unexpected results.
     ```

4. **Limited Use Cases**:
   - The primary use case for unsigned integers is when you need to represent quantities that are inherently non-negative, like sizes or counts (e.g., array indices). However, even in these cases, signed integers are often preferred to avoid the issues mentioned above.

5. **Code Portability and Readability**:
   - The behavior of unsigned integers might vary slightly between different platforms and compilers. Using signed integers can make the code more predictable and easier to understand for developers, especially those who are not deeply familiar with unsigned integer behavior.
   
6. **Error Handling Complexity**:
   - Handling errors becomes more complex when using unsigned integers, especially in cases where negative values might indicate an error or special condition. For example, returning `-1` from a function to indicate an error becomes problematic if the function returns an unsigned type.

### When to Use Unsigned Integers

Despite the potential pitfalls, there are still situations where unsigned integers are appropriate:

1. **Bit Manipulation**:
   - When performing low-level bitwise operations, unsigned integers can be useful to avoid sign extension issues that occur with signed integers.

2. **Memory and Performance Optimization**:
   - In performance-critical code where every bit counts, using unsigned integers might be justified to maximize the range of values without increasing memory usage.

3. **Domain-Specific Applications**:
   - In certain mathematical computations or applications that inherently require non-negative numbers (e.g., cryptography), unsigned integers might be the natural choice.

### Best Practices

- **Prefer Signed Integers for General Use**: For most programming tasks, signed integers are safer and less error-prone.
- **Explicit Type Conversions**: When mixing signed and unsigned integers, use explicit type conversions to avoid unintended behavior.
- **Consider the Context**: Use unsigned integers only when the problem domain clearly requires non-negative values, and be aware of the potential pitfalls.

In summary, while unsigned integers have their uses, they come with risks that can lead to subtle and hard-to-debug errors. For most general-purpose programming, it is safer to stick with signed integers unless there's a compelling reason to use unsigned types.