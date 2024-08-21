- When individual bits of an object are used as Boolean values, the bits are called **bit flags**.
- To define a set of bit flags, we’ll typically use an unsigned integer of the appropriate size (8 bits, 16 bits, 32 bits, etc… depending on how many flags we have), or std::bitset.

```cpp
#include <bitset> // for std::bitset

std::bitset<8> mybitset {}; // 8 bits in size means room for 8 flags
```