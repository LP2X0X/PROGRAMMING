- The **char** data type was designed to hold a single `character`. A **character** can be a single letter, number, symbol, or whitespace.
```ad-note
Character literals are always placed between single quotes (e.g. ‘g’, ‘1’, ‘ ‘).
```

```ad-important
The char data type is an integral type, meaning the underlying value is stored as an integer. Similar to how a Boolean value `0` is interpreted as `false` and non-zero is interpreted as `true`, the integer stored by a `char` variable are intepreted as an `ASCII character`.
```

---
- `char16_t` and `char32_t` were added to C++11 to provide explicit support for 16-bit and 32-bit Unicode characters. These char types have the same size as `std::uint_least16_t` and `std::uint_least32_t` respectively (but are distinct types). `char8_t` was added in C++20 to provide support for 8-bit Unicode (UTF-8). It is a distinct type that uses the same representation as `unsigned char`.

---
In C++, there are three distinct character types: `char`, `signed char`, and `unsigned char`. Each type occupies the same amount of memory but serves different purposes in terms of how the values are interpreted. Here’s a breakdown of their sizes and differences:

### 1. **Size of Character Types**

The size of all three character types is always 1 byte (or 8 bits) in C++. This is guaranteed by the C++ standard, which specifies that the size of a `char` is always 1 byte, and this byte is the smallest addressable unit of memory. However, the number of bits in a byte can vary across different systems, but it is usually 8 bits.

- **`char`**: 1 byte
- **`signed char`**: 1 byte
- **`unsigned char`**: 1 byte

### 2. **Ranges of Each Character Type**

While all three character types share the same memory size, their ranges differ because of how they represent values:

- **`char`**:
  - The signedness of `char` is implementation-defined, meaning it can be either signed or unsigned depending on the compiler.
  - On systems where `char` is signed, it typically ranges from `-128` to `127`.
  - On systems where `char` is unsigned, it ranges from `0` to `255`.

- **`signed char`**:
  - Always represents signed values.
  - Range: `-128` to `127` (on most systems with 8-bit bytes).

- **`unsigned char`**:
  - Always represents unsigned values.
  - Range: `0` to `255`.

### 3. **Use Cases for Each Type**

- **`char`**: 
  - Used for character data, such as representing ASCII or other character encodings.
  - The signedness being implementation-defined can lead to different behavior across platforms, so it's not typically used for numeric operations.

- **`signed char`**: 
  - Explicitly indicates that the data can have negative values.
  - Useful for storing small signed integers.

- **`unsigned char`**:
  - Commonly used for raw binary data, byte manipulation, and cases where the values should only be non-negative.
  - Suitable for tasks like representing pixel data or working with file I/O.

### Summary

- **Size**: All character types (`char`, `signed char`, `unsigned char`) are 1 byte in size in C++.
- **Range**:
  - `char`: Either `-128` to `127` (signed) or `0` to `255` (unsigned), depending on the compiler.
  - `signed char`: Always `-128` to `127`.
  - `unsigned char`: Always `0` to `255`. 

The primary difference lies in how the stored values are interpreted rather than in the memory they occupy.