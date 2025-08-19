---
tags:
  - cpp
  - note
  - fundamental
---

## 📘 Numeral Systems in C++

### 1. What Are Numeral Systems?

A **numeral system** defines the set of _digits_ and rules used to represent numbers. C++ supports **four** main systems:

1. **Decimal** (base 10) — digits `0–9` (the default in C++)
    
2. **Binary** (base 2) — digits `0–1`
    
3. **Octal** (base 8) — digits `0–7`
    
4. **Hexadecimal** (base 16) — digits `0–9` and `A–F` ([learncpp.com](https://www.learncpp.com/cpp-tutorial/numeral-systems-decimal-binary-hexadecimal-and-octal/?utm_source=chatgpt.com "5.3 — Numeral systems (decimal, binary, hexadecimal, and octal)"))
    

---

### 2. Decimal (Base 10)

- Uses digits `0` through `9`.
    
- Example:
    
    ```cpp
    int x { 12 };  // default is decimal
    ```
    
- Counting: 0, 1, 2, …, 9, 10, 11, …
    

---

### 3. Binary (Base 2)

- Only digits `0` and `1`.
    
- Often grouped in 4s with apostrophes for readability: `0b1101'0100`.
    
- Counting: 0, 1, 10 (2), 11 (3), 100 (4), 101 (5), … ([learncpp.com](https://www.learncpp.com/cpp-tutorial/numeral-systems-decimal-binary-hexadecimal-and-octal/?utm_source=chatgpt.com "5.3 — Numeral systems (decimal, binary, hexadecimal, and octal)"))
    
- Use the `0b` prefix (since C++14):
    
    ```cpp
    int b { 0b1101 };  // binary literal
    ```
    

---

### 4. Octal (Base 8)

- Digits `0–7`, no `8` or `9`.
    
- Count: 0, 1, 2, …, 7, 10 (8), 11 (9), … ([learncpp.com](https://www.learncpp.com/cpp-tutorial/numeral-systems-decimal-binary-hexadecimal-and-octal/?utm_source=chatgpt.com "5.3 — Numeral systems (decimal, binary, hexadecimal, and octal)"))
    
- To write in octal, prefix with `0`:
    
    ```cpp
    int o { 012 };  // octal literal 12 → decimal 10
    ```
    
- **Recommendation**: avoid octal unless absolutely necessary ([learncpp.com](https://www.learncpp.com/cpp-tutorial/numeral-systems-decimal-binary-hexadecimal-and-octal/?utm_source=chatgpt.com "5.3 — Numeral systems (decimal, binary, hexadecimal, and octal)"))
    

---

### 5. Hexadecimal (Base 16)

- Digits `0–9`, plus letters `A–F` (or lowercase) for `10–15`.
    
- Count: 0, 1, …, 9, A (10), B (11), …, F (15), 10 (16), … ([learncpp.com](https://www.learncpp.com/cpp-tutorial/numeral-systems-decimal-binary-hexadecimal-and-octal/?utm_source=chatgpt.com "5.3 — Numeral systems (decimal, binary, hexadecimal, and octal)"))
    
- Prefix with `0x`:
    
    ```cpp
    int h { 0xF };  // hex literal: F → decimal 15
    ```
    
- Widely used to represent binary data in a human-readable way — 1 hex digit = 4 bits ([en.wikipedia.org](https://en.wikipedia.org/wiki/Hexadecimal?utm_source=chatgpt.com "Hexadecimal"))
    

---

### 6. Comparative Table

|Decimal|Binary|Octal|Hexadecimal|
|:-:|:-:|:-:|:-:|
|0|0|0|0|
|1|1|1|1|
|2|10|2|2|
|3|11|3|3|
|4|100|4|4|
|5|101|5|5|
|6|110|6|6|
|7|111|7|7|
|8|1000|10|8|
|9|1001|11|9|
|10|1010|12|A|
|15|1111|17|F|
|16|10000|20|10|

---

### 7. Naming & Pronunciation

- Use **decimal names** (“ten”, “eleven”) only in base 10.
    
- In non-decimal systems, use **digit-by-digit** naming:
    
    - e.g. binary `101` = “one-zero-one,” not “one hundred and one” ([learncpp.com](https://www.learncpp.com/cpp-tutorial/numeral-systems-decimal-binary-hexadecimal-and-octal/?utm_source=chatgpt.com "5.3 — Numeral systems (decimal, binary, hexadecimal, and octal)"))
        

---

### 8. Usage Recommendations

- **Decimal**: default human-readable values.
- **Binary** (`0b…`): great for bitwise flags, low-level operations, and clarity.
- **Hexadecimal** (`0x…`): excellent for compact representation of bytes and memory addresses.
- **Octal** (`0…`): rarely used — avoid if possible. ([learncpp.com](https://www.learncpp.com/cpp-tutorial/numeral-systems-decimal-binary-hexadecimal-and-octal/?utm_source=chatgpt.com "5.3 — Numeral systems (decimal, binary, hexadecimal, and octal)"), [learncpp.com](https://www.learncpp.com/cpp-tutorial/bit-flags-and-bit-manipulation-via-stdbitset/?utm_source=chatgpt.com "O.1 — Bit flags and bit manipulation via std::bitset - Learn C++"))
    

---

### 9. C++ Literal Prefixes

- `0b…` → binary (since C++14)
- `0…` → octal
- `0x…` → hexadecimal
- No prefix → decimal

---

### 10. Why This Matters

Understanding numeral systems is crucial for:

- Parsing and printing data in different bases
- Bit manipulation
- Interpreting memory addresses or binary data in a readable format

---

### 11. Outputting values in decimal, octal, or hexadecimal

By default, C++ outputs values in decimal. However, you can change the output format via use of the `std::dec`, `std::oct`, and `std::hex` I/O manipulators:

```cpp
#include <iostream>

int main()
{
    int x { 12 };
    std::cout << x << '\n'; // decimal (by default)
    std::cout << std::hex << x << '\n'; // hexadecimal
    std::cout << x << '\n'; // now hexadecimal
    std::cout << std::oct << x << '\n'; // octal
    std::cout << std::dec << x << '\n'; // return to decimal
    std::cout << x << '\n'; // decimal

    return 0;
}
```

This prints:

```ad-Answer
12
c
c
14
12
12
```

```ad-note
Once applied, the I/O manipulator remains set for future output until it is changed again.
```