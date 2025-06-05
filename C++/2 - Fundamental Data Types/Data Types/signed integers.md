---
tags: cpp, datatype, fundamental
---

- By default, integers in C++ are **signed**, which means the number’s sign is stored as part of the value. So a **signed [[Integer|integer]]** can hold both positive and negative numbers (and 0).

|Type|Minimum Size|Note|
|---|---|---|
|short int|16 bits||
|int|16 bits|Typically 32 bits on modern architectures|
|long int|32 bits||
|long long int|64 bits||

- For signed integers, integer overflow will result in **undefined behavior**.

---

### Defining signed integers
- Here is the preferred way to define the four types of signed integers:

```cpp
short s;      // prefer "short" instead of "short int"
int i;
long l;       // prefer "long" instead of "long int"
long long ll; // prefer "long long" instead of "long long int"
```

```ad-note
Prefer the shorthand types that do not use the `int` suffix or `signed` prefix.
```
