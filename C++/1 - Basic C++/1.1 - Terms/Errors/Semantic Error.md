---
tags:
  - cpp
  - term
  - fundamental
---

A "semantic error" in C++ means that your code compiles successfully and may even run, but it does not behave the way you intended. In other words, the syntax is correct, but the logic or meaning (semantics) is incorrect.

---

## ❓ What is a Semantic Error?

🚫 Examples of semantic errors include:

- Using the wrong variable in a calculation
- Off-by-one errors in loops
- Mistakes in logic or conditions
- Performing valid C++ operations that lead to incorrect results

The compiler won’t catch these errors because the code is valid C++, but the program does the wrong thing.

---

## 🔍 Examples of Common Semantic Errors

---

### 1. 🧮 Wrong Variable Used in a Calculation

```cpp
int a = 10;
int b = 20;
int sum = a + a;  // 🧨 Forgot to use b
```
✅ Fix:
```cpp
int sum = a + b;
```

---

### 2. 🔄 Off-by-One Error in a Loop

```cpp
for (int i = 0; i <= 5; ++i) {  // Loops 6 times instead of 5
    cout << i << " ";
}
```
✅ Fix:
```cpp
for (int i = 0; i < 5; ++i) {
    cout << i << " ";
}
```

---

### 3. ❌ Assignment Instead of Comparison

```cpp
if (x = 5) {  // Semantically incorrect: assigns 5 to x
    // Always true
}
```
✅ Fix:
```cpp
if (x == 5) {
    // Proper comparison
}
```

---

### 4. 💡 Misunderstanding Operator Precedence

```cpp
int result = 1 + 2 * 3;  // = 7, not 9
```
✅ Fix (if that's what you intended):
```cpp
int result = (1 + 2) * 3;  // = 9
```

---

### 5. 🧠 Incorrect Use of && / ||

```cpp
if (score > 0 || score < 100) {  // Always true for any number
```
✅ Fix:
```cpp
if (score > 0 && score < 100) {  // Number is in range 1-99
```

---

### 6. 🗃️ Using Integer Division Without Realizing

```cpp
int a = 5;
int b = 2;
float result = a / b;  // result = 2.0 (not 2.5)
```
✅ Fix:
```cpp
float result = static_cast<float>(a) / b;  // result = 2.5
```

---

## 🛠️ How to Find and Fix Semantic Errors

Since the compiler won’t help with semantic errors, you need to:

1. 👀 Review your logic: Go through the code line-by-line and make sure it matches your intention.
2. 🧪 Test thoroughly: Try different input values.
3. 🐞 Use a debugger: Step through the code to watch variable values.
4. 🪵 Add print/log statements: Print key variables and values.
5. ✅ Write unit tests: Helps reproduce and isolate logic errors.

---

## 🛑 Summary

| Error Type        | Example                             | Detected by Compiler? | Fix at Compile Time? |
|-------------------|-------------------------------------|------------------------|-----------------------|
| Syntax Error      | Missing semicolon                   | ✅ Yes                 | ✅ Yes                |
| Linker Error      | Missing function definition          | ✅ Yes (link stage)    | ✅ Yes                |
| Semantic Error    | Wrong logic but compiles fine        | ❌ No                  | ❌ No                 |
