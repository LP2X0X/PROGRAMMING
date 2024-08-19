- The **char** data type was designed to hold a single `character`. A **character** can be a single letter, number, symbol, or whitespace.
```ad-note
Character literals are always placed between single quotes (e.g. ‘g’, ‘1’, ‘ ‘).
```

---
- `char16_t` and `char32_t` were added to C++11 to provide explicit support for 16-bit and 32-bit Unicode characters. These char types have the same size as `std::uint_least16_t` and `std::uint_least32_t` respectively (but are distinct types). `char8_t` was added in C++20 to provide support for 8-bit Unicode (UTF-8). It is a distinct type that uses the same representation as `unsigned char`.