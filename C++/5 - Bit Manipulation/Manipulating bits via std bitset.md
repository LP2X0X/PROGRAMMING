- std::bitset provides 4 key member functions that are useful for doing bit manipulation:
	- test() allows us to query whether a bit is a 0 or 1.
	- set() allows us to turn a bit on (this will do nothing if the bit is already on).
	- reset() allows us to turn a bit off (this will do nothing if the bit is already off).
	- flip() allows us to flip a bit value from a 0 to a 1 or vice versa.

```ad-note
Each of these functions takes the position of the bit we want to operate on as their only argument.
```

````ad-example
```cpp
#include <bitset>
#include <iostream>

int main()
{
    std::bitset<8> bits{ 0b0000'0101 }; // we need 8 bits, start with bit pattern 0000 0101
    bits.set(3);   // set bit position 3 to 1 (now we have 0000 1101)
    bits.flip(4);  // flip bit 4 (now we have 0001 1101)
    bits.reset(4); // set bit 4 back to 0 (now we have 0000 1101)

    std::cout << "All the bits: " << bits<< '\n';
    std::cout << "Bit 3 has value: " << bits.test(3) << '\n';
    std::cout << "Bit 4 has value: " << bits.test(4) << '\n';

    return 0;
}
````

````ad-tip
Given our bits names can help make our code more readable:
```cpp
#include <bitset>
#include <iostream>

int main()
{
    [[maybe_unused]] constexpr int  isHungry   { 0 };
    [[maybe_unused]] constexpr int  isSad      { 1 };
    [[maybe_unused]] constexpr int  isMad      { 2 };
    [[maybe_unused]] constexpr int  isHappy    { 3 };
    [[maybe_unused]] constexpr int  isLaughing { 4 };
    [[maybe_unused]] constexpr int  isAsleep   { 5 };
    [[maybe_unused]] constexpr int  isDead     { 6 };
    [[maybe_unused]] constexpr int  isCrying   { 7 };

    std::bitset<8> me{ 0b0000'0101 }; // we need 8 bits, start with bit pattern 0000 0101
    me.set(isHappy);      // set bit position 3 to 1 (now we have 0000 1101)
    me.flip(isLaughing);  // flip bit 4 (now we have 0001 1101)
    me.reset(isLaughing); // set bit 4 back to 0 (now we have 0000 1101)

    std::cout << "All the bits: " << me << '\n';
    std::cout << "I am happy: " << me.test(isHappy) << '\n';
    std::cout << "I am laughing: " << me.test(isLaughing) << '\n';

    return 0;
}
````