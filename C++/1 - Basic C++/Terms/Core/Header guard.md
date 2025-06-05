---
tags: cpp, term, fundamental
---

- **Header guards** are [[Conditional compilation|conditional compilation directives]] that take the following form:
```cpp
#ifndef SOME_UNIQUE_NAME_HERE
#define SOME_UNIQUE_NAME_HERE

// your declarations (and certain types of definitions) here

#endif
```

- When this header is \#included, the preprocessor will check whether _SOME_UNIQUE_NAME_HERE_ has been previously defined in this translation unit.
	- If this is the first time we’re including the header, _SOME_UNIQUE_NAME_HERE_ will not have been defined. Consequently, it \#defines _SOME_UNIQUE_NAME_HERE_ and includes the contents of the file.
	- If the header is included again into the same file, _SOME_UNIQUE_NAME_HERE_ will already have been defined from the first time the contents of the header were included, and the contents of the header will be ignored (thanks to the \#ifndef).

```ad-tip
All of your [[Header file|header files]] should have header guards on them.
```

```ad-important
Header guard do not prevent the contents of a header from being included into multiple different code files. This is why you should not always define something in the header file.
```
---

### \#pragma once
- `#pragma once` is a [[Preprocessor Directive|preprocessor directive]] in C++ that ensures a header file is included only once in a single compilation, preventing multiple inclusions of the same header file. This is particularly useful to avoid issues like redefinition errors or bloated compile times caused by repeatedly including the same file.
- In modern C++ development, `#pragma once` is often preferred due to its simplicity, but include guards remain a more portable solution.

#### How It Works

When you include a header file in multiple places (directly or indirectly), the compiler might attempt to include the same file multiple times. `#pragma once` tells the compiler to include that file only once per translation unit, regardless of how many times it is included.

#### Example Usage

```cpp
// myheader.h

#pragma once

// Declarations and definitions
class MyClass {
public:
    void myFunction();
};
```

#### Advantages of `#pragma once`
- **Simplicity**: Easier and less error-prone to write compared to traditional include guards.
- **Performance**: In some compilers, `#pragma once` can be faster because it is a directive for the preprocessor to skip the file without even opening it after the first inclusion.

#### Disadvantages of `#pragma once`
- **Portability**: It is not part of the C++ standard, though it is widely supported by most modern compilers (e.g., GCC, Clang, MSVC). However, older or less common compilers might not support it.
