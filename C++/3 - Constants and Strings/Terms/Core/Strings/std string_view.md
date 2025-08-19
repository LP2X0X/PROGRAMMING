---
tags:
  - cpp
  - term
  - fundamental
---

- To address the issue with `std::string` being expensive to initialize (or copy), C++17 introduced `std::string_view` (which lives in the <string_view> header). `std::string_view` provides read-only access to an *existing* string (a C-style string, a `std::string`, or another `std::string_view`) without making a copy. 

```ad-note
A `std::string_view` may or may not be null-terminated.
```

```cpp
#include <iostream>
#include <string>
#include <string_view>

int main()
{
    std::string_view s1 { "Hello, world!" }; // initialize with C-style string literal
    std::cout << s1 << '\n';

    std::string s{ "Hello, world!" };
    std::string_view s2 { s };  // initialize with std::string
    std::cout << s2 << '\n';

    std::string_view s3 { s2 }; // initialize with std::string_view
    std::cout << s3 << '\n';

    return 0;
}
```

- `std::string_view` will not implicitly convert to `std::string`. This is to prevent accidentally passing a `std::string_view` argument to a `std::string` parameter, and inadvertently making an expensive copy where such a copy may not be required.

---

### View modification functions
- Because `std::string_view` is a view, it contains functions that let us <u>permanently</u> modify our view. This does not modify the string being viewed in any way, just the view itself:
	- The `remove_prefix()` *member function* removes characters from the left side of the view.
	- The `remove_suffix()` *member function* removes characters from the right side of the view.

```cpp
#include <iostream>
#include <string_view>

int main()
{
	std::string_view str{ "Peach" };
	std::cout << str << '\n';

	// Remove 1 character from the left side of the view
	str.remove_prefix(1);
	std::cout << str << '\n';

	// Remove 2 characters from the right side of the view
	str.remove_suffix(2);
	std::cout << str << '\n';

	str = "Peach"; // reset the view
	std::cout << str << '\n';

	return 0;
}
```

---

### constexpr std::string_view

- `std::string_view` has full support for constexpr:

```cpp
#include <iostream>
#include <string_view>

int main()
{
    constexpr std::string_view s{ "Hello, world!" }; // s is a string symbolic constant
    std::cout << s << '\n'; // s will be replaced with "Hello, world!" at compile-time

    return 0;
}
```

---

### Cautions

- A `std::string_view` that is viewing a string that has been destroyed is sometimes called a **dangling** view.
- When a `std::string` is modified, all views into that `std::string` are **invalidated**, meaning those views are now invalid.
- Do not initialize a `std::string_view` with a [[std string literal|`std::string` literal]], as this will leave the `std::string_view` dangling.

---

#### Return std::string_view
- There are two main cases where a `std::string_view` can be returned safely. First, because [[C-style string literal|C-style string literals]] exist for the entire program, it’s fine (and useful) to return C-style string literals from a function that has a return type of `std::string_view`.

```cpp
#include <iostream>
#include <string_view>

std::string_view getBoolName(bool b)
{
    if (b)
        return "true";  // return a std::string_view viewing "true"

    return "false"; // return a std::string_view viewing "false"
} // "true" and "false" are not destroyed at the end of the function

int main()
{
    std::cout << getBoolName(true) << ' ' << getBoolName(false) << '\n'; // ok

    return 0;
}
```

- Second, it is generally okay to return a function parameter of type `std::string_view`:

```cpp
#include <iostream>
#include <string>
#include <string_view>

std::string_view firstAlphabetical(std::string_view s1, std::string_view s2)
{
    return s1 < s2 ? s1: s2; // uses operator?: (the conditional operator)
}

int main()
{
    std::string a { "World" };
    std::string b { "Hello" };

    std::cout << firstAlphabetical(a, b) << '\n'; // prints "Hello"

    return 0;
}
```

#### View modification functions
- Because `std::string_view` is a view, it contains functions that let us modify our view by “closing the curtains”. This does not modify the string being viewed in any way, just the view itself:
	- The `remove_prefix()` member function removes characters from the left side of the view.
	- The `remove_suffix()` member function removes characters from the right side of the view.

```cpp
#include <iostream>
#include <string_view>

int main()
{
	std::string_view str{ "Peach" };
	std::cout << str << '\n';

	// Remove 1 character from the left side of the view
	str.remove_prefix(1);
	std::cout << str << '\n';

	// Remove 2 characters from the right side of the view
	str.remove_suffix(2);
	std::cout << str << '\n';

	str = "Peach"; // reset the view
	std::cout << str << '\n';

	return 0;
}
```

```ad-note
Unlike real curtains, once `remove_prefix()` and `remove_suffix()` have been called, the only way to reset the view is by reassigning the source string to it again.
```

#### `std::string_view` may or may not be null-terminated
- Consider the string “snowball”, which is null-terminated (because it is a C-style string literal, which are always null-terminated). If a `std::string_view` views the whole string, then it is viewing a null-terminated string. However, if `std::string_view` is only viewing the substring “now”, then that substring is not null-terminated (the next character is a ‘b’).
  
--- 

### **Why We Need `std::string_view` Literal**

1. **Performance Optimization**:
   - **Avoids Copying**: When you use `std::string_view`, you avoid copying the string data. A `std::string` involves heap allocation and copying the contents of the string, which can be expensive, especially for large strings. `std::string_view`, on the other hand, simply points to an existing string's data, making it much faster.
   
   Example:
   ```cpp
   #include <string_view>
   using namespace std::literals;

   std::string_view sv = "Hello, World!"sv;
   ```

   - Here, `"Hello, World!"sv` creates a `std::string_view` that references the string literal directly without copying the data. This is more efficient than using a `std::string` when you only need to read or inspect the string.

2. **Reduced Memory Usage**:
   - **No Heap Allocation**: `std::string_view` does not allocate memory on the heap. It only stores a pointer to the string data and the length of the string. This reduces memory usage compared to `std::string`, which requires heap allocation for its data.
   
   - **Efficient Passing**: `std::string_view` is lightweight, usually consisting of just a pointer and a size, making it efficient to pass around by value in function arguments, especially when you need to pass large strings or when the function only needs to read the string data.

3. **Safe and Easy-to-Use Non-Owning Reference**:
   - **Non-Owning**: `std::string_view` is a non-owning reference, meaning it doesn’t manage the lifetime of the string data it points to. This makes it ideal for scenarios where you need a temporary reference to a string without taking ownership.

   - **Range-Based Operations**: You can use `std::string_view` in range-based operations like iterating over characters, slicing, or finding substrings, all without copying the data.

   Example:
   ```cpp
   void print(std::string_view sv) {
       std::cout << sv << std::endl;
   }

   int main() {
       print("Hello, World!"sv);  // No std::string copy, directly uses the string_view
   }
   ```

4. **Literal Support for Convenience**:
   - **Ease of Use**: The `sv` suffix allows you to easily create a `std::string_view` from a string literal. This is especially convenient when writing functions that accept `std::string_view` parameters, as you can simply pass literals with the `sv` suffix without needing to explicitly create a `std::string_view` object.

5. **Compile-Time Constants**:
   - **Compile-Time Efficiency**: Since string literals are compile-time constants, using `std::string_view` with the `sv` suffix allows you to reference these constants efficiently at runtime without the overhead of creating and copying `std::string` objects.

### **When to Use `std::string_view` vs. `std::string`**

- **Use `std::string_view`** when:
  - You need a non-owning, read-only view of a string.
  - Performance is critical, and you want to avoid unnecessary copying and memory allocation.
  - You are working with large strings and need to pass them around efficiently.
  - You need to process string literals or substrings without modifying them.

- **Use `std::string`** when:
  - You need to own and manage the string's memory (e.g., when the string's lifetime needs to be independent of the original string).
  - You need to modify the string data.
  - You need dynamic string operations like concatenation, mutation, or reallocation.

### **Summary**

The `std::string_view` literal (`sv`) provides a powerful tool for efficient and safe string manipulation by enabling non-owning references to strings. This is particularly useful for performance-critical applications where avoiding unnecessary memory allocations and copies is important. The `sv` suffix simplifies the creation of `std::string_view` objects, making it easier to write efficient and concise code.