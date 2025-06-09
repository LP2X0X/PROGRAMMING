---
tags: cpp, term, fundamental
---

- In programming, a **string** is a collection of sequential characters used to represent text (such as names, words, and sentences). Such strings are often called **C strings** or **C-style strings**, as they are inherited from the C-language.

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello, world!"; // "Hello, world!" is a string literal.
    return 0;
}
```

- There are two non-obvious things worth knowing about C-style string literals:
	- All C-style string literals have an implicit null terminator. 
		- Consider a string such as `"hello"`. While this C-style string appears to only have five characters, it actually has six: `'h'`, `'e'`, `'l`‘, `'l'`, `'o'`, and `'\0'` (a character with ASCII code 0). This trailing ‘\0’ character is a special character called a **null terminator**, and it is used to indicate the end of the string. A string that ends with a null terminator is called a **null-terminated string**.
	- Unlike most other literals (which are values, not objects), C-style string literals are const objects that are created at the start of the program and are guaranteed to exist for the entirety of the program.