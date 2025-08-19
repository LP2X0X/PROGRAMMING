---
tags:
  - cpp
  - term
  - fundamental
---

- A **literal** (also known as a literal constant) is a fixed value that has been inserted directly into the source code.

```ad-note
Literals are sometimes called **literal constants** because their meaning cannot be redefined (`5` always means the integral value 5).
```

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
