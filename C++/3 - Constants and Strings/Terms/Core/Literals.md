- A **literal** (also known as a literal constant) is a fixed value that has been inserted directly into the source code.

---

### Type of literal
| Literal value        | Examples        | Default literal type | Note |
| -------------------- | --------------- | -------------------- | ---- |
| integer value        | 5, 0, -3        | int                  |      |
| boolean value        | true, false     | bool                 |      |
| floating point value | 1.2, 0.0, 3.4   | double (not float!)  |      |
| character            | ‘a’, ‘\n’       | char                 |      |
| C-style string       | “Hello, world!” | const char[14]       |      |

### Literal suffixes
| Data type      | Suffix                                 | Meaning                                   |
| -------------- | -------------------------------------- | ----------------------------------------- |
| integral       | u or U                                 | unsigned int                              |
| integral       | l or L                                 | long                                      |
| integral       | ul, uL, Ul, UL, lu, lU, Lu, LU         | unsigned long                             |
| integral       | ll or LL                               | long long                                 |
| integral       | ull, uLL, Ull, ULL, llu, llU, LLu, LLU | unsigned long long                        |
| integral       | z or Z                                 | The signed version of std::size_t (C++23) |
| integral       | uz, uZ, Uz, UZ, zu, zU, Zu, ZU         | std::size_t (C++23)                       |
| floating point | f or F                                 | float                                     |
| floating point | l or L                                 | long double                               |
| string         | s                                      | std::string                               |
| string         | sv                                     | std::string_view                          |

---

### String literals
- In programming, a **string** is a collection of sequential characters used to represent text (such as names, words, and sentences).
- There are two non-obvious things worth knowing about C-style string literals:
	- All C-style string literals have an implicit null terminator. Consider a string such as `"hello"`. While this C-style string appears to only have five characters, it actually has six: `'h'`, `'e'`, `'l`‘, `'l'`, `'o'`, and `'\0'` (a character with ASCII code 0). This trailing ‘\0’ character is a special character called a **null terminator**, and it is used to indicate the end of the string. A string that ends with a null terminator is called a **null-terminated string**.
	- Unlike most other literals (which are values, not objects), C-style string literals are const objects that are created at the start of the program and are guaranteed to exist for the entirety of the program.