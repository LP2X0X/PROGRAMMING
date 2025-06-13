---
tags: cpp, datatype, fundamental
---

- Since [[Integer#Why isn’t the size of the integer types fixed?|normal integer types does not have fixed memory]], C++11 provides an alternate set of integer types that are guaranteed to be the same size on any architecture. Because the size of these integers is fixed, they are called **fixed-width integers**.
- The fixed-width integers actually don’t define new types -- they are just aliases for existing integral types with the desired size.
- The fixed-width integers are defined (in the \<cstdint> - c standard integer - header) as follows:

| Name          | Fixed Size      | Fixed Range                                             | Notes                                                          |
| ------------- | --------------- | ------------------------------------------------------- | -------------------------------------------------------------- |
| std::int8_t   | 1 byte signed   | -128 to 127                                             | Treated like a signed char on many systems. See note below.    |
| std::uint8_t  | 1 byte unsigned | 0 to 255                                                | Treated like an unsigned char on many systems. See note below. |
| std::int16_t  | 2 byte signed   | -32,768 to 32,767                                       |                                                                |
| std::uint16_t | 2 byte unsigned | 0 to 65,535                                             |                                                                |
| std::int32_t  | 4 byte signed   | -2,147,483,648 to 2,147,483,647                         |                                                                |
| std::uint32_t | 4 byte unsigned | 0 to 4,294,967,295                                      |                                                                |
| std::int64_t  | 8 byte signed   | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 |                                                                |
| std::uint64_t | 8 byte unsigned | 0 to 18,446,744,073,709,551,615                         |                                                                |

```ad-note
Pay attention to the [[The _t suffix|`_t`]] suffix.
```

```ad-important
Due to an oversight in the C++ specification, modern compilers typically treat `std::int8_t` and `std::uint8_t` the same as `signed char` and `unsigned char` respectively.
When they are treated as a char, **the input routines interpret our input as a sequence of characters**, not as an integer.
```