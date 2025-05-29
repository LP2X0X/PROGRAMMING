---
tags: cpp, term, fundamental
---

A **link error** in C++ typically refers to issues that arise during the linking stage of [[Compilation Process#4. **Linking**|the compilation process]]. After your source code is compiled into object files, the linker combines those object files and resolves references between them. If something goes wrong at this stage, you get a link error.

Here are common causes of link errors in C++ and how to fix them:

---

### 🔗 Common Link Errors

#### 1. **Undefined Reference**
Example:

```cpp
// main.cpp
void foo();

int main() {
    foo();  // Linker error: undefined reference to `foo()`
}
```

Fix:

Make sure the function `foo()` is defined:

```cpp
// foo.cpp
#include <iostream>
void foo() {
    std::cout << "Hello from foo()" << std::endl;
}
```

Compile and link both files:

```bash
g++ main.cpp foo.cpp -o myprogram
```

---

#### 2. **Multiple Definition Errors**
Example:

```cpp
// a.cpp
int value = 5;
// b.cpp
int value = 5;  // Linker error: multiple definition of `value`
```

Fix:

Use `extern` in header files and define the variable in only one `.cpp` file:

```cpp
// header.h
extern int value;  // declaration

// a.cpp
#include "header.h"
int value = 5;  // definition
```

---

#### 3. **Missing Object or Library File**
Linker can't find the implementation needed, perhaps you're missing a library or object file:

```bash
g++ main.cpp -o myprog
```

If `main.cpp` calls functions from `utils.cpp` but you didn’t include it in compilation, the linker will complain.

Fix:

Compile with all needed files:

```bash
g++ main.cpp utils.cpp -o myprog
```

---

#### 4. **Incorrect Function Signatures**
Mismatch between declaration and definition.

Header:

```cpp
void printMessage(std::string msg);
```

Implementation:

```cpp
void printMessage(char* msg) { /* ... */ }  // Link error
```

Fix:

Function declarations and definitions must match exactly—in name, return type, and parameter types.

---

#### 5. **Template Link Errors**
Templates need to have implementation in the header because the compiler needs to see the source when instantiating them.

Bad:

```cpp
// my_template.h
template <typename T>
T add(T a, T b);
```

// my_template.cpp
```cpp
template <typename T>
T add(T a, T b) { return a + b; }
```

Good:

Put both declaration and definition in the header:

```cpp
// my_template.h
template <typename T>
T add(T a, T b) { return a + b; }
```

---

### 🛠️ Tips to Diagnose Link Errors

- Use the full command line with all source files.
- Check for typos in function names or signatures.
- Ensure template definitions are visible where used.
- For external libraries, use `-l` and `-L` flags (e.g. `-lboost_system`).
- Use `nm` or `objdump` to inspect object files.
- Use `g++ -c` to compile to object files and `g++ *.o -o prog` to link manually.