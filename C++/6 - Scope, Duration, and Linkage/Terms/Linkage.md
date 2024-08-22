In C++, linkage refers to the visibility of symbols (such as variables, functions, and objects) across different translation units, which are typically individual source files. Linkage determines whether a symbol can be accessed from other translation units or is limited to the translation unit where it is defined. There are three main types of linkage in C++: **external linkage**, **internal linkage**, and **no linkage**.

### 1. **External Linkage**
   - **Characteristics**:
     - Symbols with external linkage **can be accessed from any translation unit in the program**.
     - Typically used for global variables, functions, and objects that need to be accessible across multiple files.
     - The default linkage for non-`const` global variables and functions.

   - **How to Achieve**:
     - Global variables and functions declared outside of any function or class automatically have external linkage unless explicitly specified otherwise.
     - `extern` keyword can be used to declare a variable or function with external linkage, indicating that the definition is provided elsewhere.

   - **Example**:
     ```cpp
     // file1.cpp
     int global_var = 42; // global_var has external linkage by default

     void foo() { /*...*/ } // foo has external linkage by default

     // file2.cpp
     extern int global_var; // Declares global_var from file1.cpp
     extern void foo();     // Declares foo from file1.cpp
     ```

### 2. **Internal Linkage**
   - **Characteristics**:
     - Symbols with internal linkage are **only visible within the translation unit where they are defined**.
     - Useful for encapsulating implementation details within a single source file, preventing access from other translation units.
     - The `static` keyword is used to give a symbol internal linkage when defined at the global scope.

```ad-note
Const and constexpr global variables have internal linkage by default (and thus don’t need the `static` keyword -- if it is used, it will be ignored).
```

   - **How to Achieve**:
     - Use the `static` keyword in front of the global variable or function definition to restrict its visibility to the translation unit.

   - **Example**:
     ```cpp
     // file1.cpp
     static int local_var = 42; // local_var has internal linkage
     static void foo() { /*...*/ } // foo has internal linkage

     // file2.cpp
     // Cannot access local_var or foo here because they have internal linkage
     ```

### 3. **No Linkage**
   - **Characteristics**:
     - Symbols with no linkage are only accessible within the scope in which they are defined.
     - Typical examples include local variables inside functions and parameters of functions.
     - Each occurrence of the symbol refers to a distinct entity.

   - **Example**:
     ```cpp
     void func(int x) {
         int local_var = 42; // local_var has no linkage
         // x also has no linkage
     }
     ```

### **Summary of Linkage Types**

| Linkage Type      | Visibility                            | Common Usage                         | Keyword/Default Behavior         |
|-------------------|---------------------------------------|--------------------------------------|----------------------------------|
| **External**      | Across all translation units          | Global variables, functions          | Default for non-`const` globals  |
| **Internal**      | Within the same translation unit      | File-local symbols                   | `static` keyword                 |
| **No Linkage**    | Within the scope of definition only   | Local variables, function parameters | No specific keyword              |

### **Linkage Examples**

```cpp
// file1.cpp
int global_var = 10;          // External linkage
static int static_var = 20;   // Internal linkage

void foo() {                  // External linkage
    int local_var = 30;       // No linkage
    static int func_static_var = 40; // No linkage (for func_static_var)
}

// file2.cpp
extern int global_var;        // Access global_var from file1.cpp
// Cannot access static_var or foo's local_var from file1.cpp
```

In summary, linkage in C++ controls the visibility and accessibility of symbols across translation units. External linkage allows symbols to be shared across multiple files, internal linkage restricts symbols to a single translation unit, and no linkage confines symbols to a specific scope. Understanding linkage is crucial for managing symbol visibility and avoiding conflicts in larger programs.