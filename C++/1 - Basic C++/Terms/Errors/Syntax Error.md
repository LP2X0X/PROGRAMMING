---
tags:
  - cpp
  - term
  - fundamental
---

A "syntax error" in C++ means there's something wrong with how your source code is written—it's not following the rules of the C++ language grammar. This prevents the compiler from understanding or translating your code.

---

## 🔧 Common C++ Syntax Errors (And Fixes)

Here are some of the most frequent syntax errors you might encounter:

---

### 1. ❌ Missing Semicolon

❌ Error:
```cpp
int main() {
    int a = 5
    return 0;
}
```

✅ Fix:
```cpp
int a = 5;
```

---

### 2. ❌ Mismatched Braces `{}` or Parentheses `()`

❌ Error:
```cpp
int main() {
    if (x > 0) {
        cout << x << endl;
    // Missing closing brace
```

✅ Fix:
```cpp
    }
```

---

### 3. ❌ Using undeclared variables or functions

❌ Error:
```cpp
int main() {
    cout << message << endl;  // 'message' and 'cout' undeclared
}
```

✅ Fix:
```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    string message = "Hello";
    cout << message << endl;
}
```

---

### 4. ❌ Assignment inside condition (wrong expectation)

❌ Error:
```cpp
if (x = 5) {  // This assigns 5 to x, doesn't check equality
    // do something
}
```

✅ Fix:
```cpp
if (x == 5) {
    // do something
}
```

---

### 5. ❌ Class or Function Definitions Missing Keywords or Semicolon

❌ Error (missing semicolon after class):
```cpp
class MyClass {
public:
    int x;
    void doSomething() {}
}  // Missing semicolon here
```

✅ Fix:
```cpp
};  // <-- required semicolon
```

---

### 6. ❌ Wrong Function Syntax

❌ Error:
```cpp
int main {
    return 0;
}
```

✅ Fix:
```cpp
int main() {
    return 0;
}
```

---

## 👀 How to Spot and Fix Syntax Errors

- Read the compiler error message! It will usually give you:
  - File name
  - Line number
  - A short description of what went wrong

- Start by looking at the first error; often other errors are side-effects.
- Check the code right before the error line — problems earlier in code can lead to confusing messages later.

---

## 🛠️ Diagnostic Tips

- Use `g++` or `clang++` with warnings:
  ```bash
  g++ -Wall -std=c++17 yourfile.cpp
  ```

- Use an IDE or code editor like VS Code, CLion, or Code::Blocks, which highlights syntax errors in real time.

- Use an online compiler to quickly test small code snippets: [https://godbolt.org](https://godbolt.org), [https://replit.com](https://replit.com)
