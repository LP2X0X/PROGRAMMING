---
tags: cpp, term, fundamental
---
- **Preprocessor directives** (often just called _directives_) are instructions that start with a _#_ symbol and end with a newline (NOT a semicolon). These directives tell the preprocessor to perform certain text manipulation tasks and are directives for the _[[Preprocessor|preprocessor]]_.
- The preprocessor **examines the code before actual compilation of code begins** and resolves all these directives before any code is actually generated by regular statements.
```ad-note
These _preprocessor directives_ extend only across a single line of code. As soon as a newline character is found, the preprocessor directive is ends. The only way a preprocessor directive can extend through more than one line is by preceding the newline character at the end of the line by a backslash (`\`).
```

---

### Common preprocessor directive
#### 🔹 1. `#include`

Includes the contents of another file (usually a header).

```cpp
#include <iostream>     // Standard library header
#include "myheader.h"   // User-defined header
```

---

#### 🔹 2. `#define`

The _\#define_ directive can be used to create a [[Macro|macro]].

```cpp
#define PI 3.14159
#define SQUARE(x) ((x) * (x))
```

---

#### 🔹 3. `#undef`

Undefines a macro.

```cpp
#undef PI
```

---

#### 🔹 4. `#ifdef`, `#ifndef`, `#endif`

The [[Conditional compilation|conditional compilation]] directives allow you to specify under what conditions something will or won’t compile. 

```cpp
#ifdef DEBUG
    std::cout << "Debugging enabled\n";
#endif

#ifndef MY_HEADER_H
#define MY_HEADER_H
// header contents
#endif
```

---

#### 🔹 5. `#if`, `#else`, `#elif`, `#endif`

Used for  [[Conditional compilation|conditional compilation]] based on [[Constant expression|constant expressions]].

```cpp
#define VERSION 2

#if VERSION == 1
    std::cout << "Version 1\n";
#elif VERSION == 2
    std::cout << "Version 2\n";
#else
    std::cout << "Unknown version\n";
#endif
```

---

#### 🔹 6. `#pragma`

Used for compiler-specific instructions (not standardized).

```cpp
#pragma once  // Non-standard but widely supported include guard
#pragma warning(disable : 4996)  // Disable specific warning (MSVC)
```