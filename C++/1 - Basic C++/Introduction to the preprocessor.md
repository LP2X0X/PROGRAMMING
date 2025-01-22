- Prior to compilation, each code (.cpp) file goes through a **preprocessing** phase. In this phase, a program called the **preprocessor** makes various changes to the text of the code file. The preprocessor does not actually modify the original code files in any way -- rather, all changes made by the preprocessor happen either temporarily in-memory or using temporary files.
- When the preprocessor has finished processing a code file, the result is called a **translation unit**. This translation unit is what is then compiled by the compiler.

### Preprocessor directives
- When the preprocessor runs, it scans through the code file (from top to bottom), looking for preprocessor directives. **Preprocessor directives** (often just called _directives_) are instructions that start with a _#_ symbol and end with a newline (NOT a semicolon). These directives tell the preprocessor to perform certain text manipulation tasks.

---

### Macro defines
- The _\#define_ directive can be used to create a [[Macro|macro]]. A **macro** is a label defined in the source code that is replaced by its value by the preprocessor before compilation.
- There are two basic types of macros: _object-like macros_, and _function-like macros_.
	- _Function-like macros_ act like functions, and serve a similar purpose. Their use is generally considered unsafe, and almost anything they can do can be done by a normal function.
	- _Object-like macros_ can be defined in one of two ways:

```cpp
#define IDENTIFIER
#define IDENTIFIER substitution_text
```
- The top definition has no substitution text, whereas the bottom one does. Because these are preprocessor directives (not statements), note that neither form ends with a semicolon.

---

### Conditional compilation
- The _[[Conditional compilation|conditional compilation]]_ preprocessor directives allow you to specify under what conditions something will or won’t compile. There are quite a few different conditional compilation directives, but the three that are used by far the most are: _#ifdef_, _#ifndef_, and _#endif_.
- The _#ifdef_ preprocessor directive allows the preprocessor to check whether an identifier has been previously _#define_d. If so, the code between the _#ifdef_ and matching _#endif_ is compiled. If not, the code is ignored.
- One more common use of conditional compilation involves using _#if 0_ to exclude a block of code from being compiled (as if it were inside a comment block):
```cpp
#include <iostream>

int main()
{
    std::cout << "Joe\n";

#if 0 // Don't compile anything starting here
    std::cout << "Bob\n";
    std::cout << "Steve\n";
#endif // until this point

    return 0;
}
```
- To temporarily re-enable code that has been wrapped in an `#if 0`, you can change the `#if 0` to `#if 1`.

--- 

### Do not \#include .cpp file
- There are number of reasons for this:
	- Doing so can cause naming collisions between source files.
	- In a large project it can be hard to avoid one definition rules (ODR) issues.
	- Any change to such a .cpp file will cause both the .cpp file and any other .cpp file that includes it to recompile, which can take a long time. Headers tend to change less often than source files.
	- It is non-conventional to do so.