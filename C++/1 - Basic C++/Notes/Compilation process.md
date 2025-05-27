---
tags: cpp, fundamental
---

```ad-note
Remember, the compiler compiles each file individually.
```

### 1. **Preprocessing**

In this initial stage, the compiler prepares the source code for compilation by handling preprocessor directives. These directives start with a `#` symbol and are processed before any actual compilation takes place.

Key tasks performed in this stage include:

- **File Inclusion**: The `#include` directive is replaced with the contents of the corresponding header files.
- **Macro Expansion**: All macros defined with `#define` are expanded wherever they are used.
- **Conditional Compilation**: Directives like `#ifdef`, `#ifndef`, `#if`, and `#endif` are evaluated to include or exclude blocks of code.
- **Comment Removal**: All comments are stripped from the source code.

The result of preprocessing is a translation unit — a single, self-contained file that is passed to the next stage.

---

### 2. **Compilation**

In the compilation stage, the preprocessed source code is translated into a lower-level representation called **assembly language**.

During this phase, the compiler performs:

- **Syntax Analysis**: Ensures the code follows the rules of C++ syntax.
- **Semantic Analysis**: Checks for meaningful use of language constructs (e.g., variable types, function calls, etc.).
- **Optimization**: Performs low- or high-level optimizations on the code to improve performance or reduce size.
- **Code Generation**: Converts the verified and optimized code into assembly instructions for the target architecture.

The output of this stage is human-readable assembly code that represents the logic of the original source file.

---

### 3. **Assembly**

The assembly stage takes the assembly code and translates it into **machine code** — the raw binary instructions that the computer's processor can execute.

This machine code is packaged into an **object file**. The object file includes:

- Compiled functions and data.
    
- Symbol tables that mark functions and variables.
    
- Unresolved references to functions or variables defined in other files (i.e., external symbols).
    

Each `.cpp` source file usually produces its own object file.

---

### 4. **Linking**

Linking is the final stage that produces a complete, standalone executable from one or more object files.

During linking:

- **Symbol Resolution**: The linker matches function and variable references in object files to their definitions. This includes both your own code and code from libraries (e.g., the C++ standard library).
- **Address Binding**: It assigns final memory addresses to all code and data sections.
- **Library Linking**: It includes the machine code of required libraries, either statically (copied into the executable) or dynamically (referenced for runtime loading).

The result is a fully linked executable file that can be run on the target system.

---

### Summary

|Stage|Description|
|---|---|
|Preprocessing|Handles `#include`, `#define`, conditional compilation, and comment removal|
|Compilation|Translates code into assembly; checks syntax and semantics|
|Assembly|Converts assembly to machine code and generates object files|
|Linking|Resolves references and produces a final executable|

---

This pipeline is foundational to how all C++ programs are built, regardless of platform or compiler. 