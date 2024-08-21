- A **bit mask** is a predefined set of bits that is used to select which specific bits will be modified by subsequent operations.
- The simplest set of bit masks is to define one bit mask for each bit position. We use 0s to mask out the bits we don’t care about, and 1s to denote the bits we want modified.

````ad-example
Example of bit masks
```cpp
constexpr std::uint8_t mask0{ 0b0000'0001 }; // represents bit 0
constexpr std::uint8_t mask1{ 0b0000'0010 }; // represents bit 1
constexpr std::uint8_t mask2{ 0b0000'0100 }; // represents bit 2
constexpr std::uint8_t mask3{ 0b0000'1000 }; // represents bit 3
constexpr std::uint8_t mask4{ 0b0001'0000 }; // represents bit 4
constexpr std::uint8_t mask5{ 0b0010'0000 }; // represents bit 5
constexpr std::uint8_t mask6{ 0b0100'0000 }; // represents bit 6
constexpr std::uint8_t mask7{ 0b1000'0000 }; // represents bit 7
````

---

### Summary

To query bit states, we use _bitwise AND_:

```cpp
if (flags & option4) ... // if option4 is set, do something
```

To set bits (turn on), we use _bitwise OR_:

```cpp
flags |= option4; // turn option 4 on.
flags |= (option4 | option5); // turn options 4 and 5 on.
```

To clear bits (turn off), we use _bitwise AND_ with _bitwise NOT_:

```cpp
flags &= ~option4; // turn option 4 off
flags &= ~(option4 | option5); // turn options 4 and 5 off
```

To flip bit states, we use _bitwise XOR_:

```cpp
flags ^= option4; // flip option4 from on to off, or vice versa
flags ^= (option4 | option5); // flip options 4 and 5
```