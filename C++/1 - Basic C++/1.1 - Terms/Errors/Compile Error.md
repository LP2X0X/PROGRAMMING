---
tags:
  - cpp
  - term
  - fundamental
---

A **compile error** in C++ occurs during [[Compilation Process#2. **Compilation**|the compilation phase]], where the compiler translates your source code into machine-readable object code. Compile errors usually stem from syntax mistakes, missing declarations, type mismatches, or other violations of the C++ language rules.

---

## 🔎 Common C++ Compile Errors (And How to Fix Them)

---

### 1. ❌ Missing Semicolon

**Code:**
```cpp
int main() {
    int x = 5  // Error! Missing semicolon
    return 0;
}
```

**Fix:**
```cpp
int x = 5;
```

---

### 2. ❌ Undeclared Identifier

**Code:**
```cpp
int main() {
    cout << "Hello, world!" << endl;  // Error: 'cout' not declared
}
```

**Fix:**
Add the proper header and use the std namespace:
```cpp
#include <iostream>
using namespace std;
```
Or:
```cpp
std::cout << "Hello, world!" << std::endl;
```

---

### 3. ❌ Mismatched Function Signature

**Code:**
```cpp
int sum(int a, int b);

int main() {
    cout << sum(3);  // Error: too few arguments
}

int sum(int a, int b) {
    return a + b;
}
```

**Fix:**
```cpp
sum(3, 4);
```

---

### 4. ❌ Conflicting Types or Return Types

**Header:**
```cpp
int getValue();
```

**Implementation:**
```cpp
float getValue() { return 3.14; }  // Error: return type mismatch with declaration
```

**Fix:** Make the return type match.

---

### 5. ❌ No Matching Function Call

**Code:**
```cpp
void print(int x);

int main() {
    print("Hello");  // Error: no matching function for call to 'print(const char*)'
}
```

**Fix:** Either change the function or the argument type.

---

### 6. 🧩 Missing Header Files

**Code:**
```cpp
int main() {
    string name = "Alice";  // Error: 'string' was not declared in this scope
}
```

**Fix:**
```cpp
#include <string>
```

---

### 7. 📦 Class Definition Errors

**Code:**
```cpp
class MyClass {
    int x
public:
    void show();
};
```

**Error:** Missing semicolon or bad syntax in the class.

**Fix:**
```cpp
int x;  // Add semicolon
```

---

### 8. 🧠 Type Mismatch

**Code:**
```cpp
int x = "hello";  // Error: cannot convert ‘const char*’ to ‘int’
```

**Fix:**
```cpp
std::string x = "hello";
```

---

### 🛠️ How to Get Help

If you're still stuck:

1. 🔍 Look at the **line number** and **exact message** from the compiler.
2. 📋 Paste the error message and code around the error.
3. 🧰 Use tools like:
   - Compiler command (e.g. `g++ -Wall -std=c++17 main.cpp`)
   - Online compilers (Godbolt, Compiler Explorer, etc.)
   - Static analyzers or IDEs (VS Code, CLion, etc.)
