- The most common method of doing floating point equality involves using a function that looks to see if two numbers are _almost_ the same. If they are “close enough”, then we call them equal. The value used to represent “close enough” is traditionally called **epsilon**. Epsilon is generally defined as a small positive number (e.g. 0.00000001, sometimes written 1e-8).
- [Donald Knuth](https://en.wikipedia.org/wiki/Donald_Knuth), a famous computer scientist, suggested the following method in his book “The Art of Computer Programming, Volume II: Seminumerical Algorithms (Addison-Wesley, 1969)”:

```cpp
#include <algorithm> // for std::max
#include <cmath>     // for std::abs

// Return true if the difference between a and b is within epsilon percent of the larger of a and b
bool approximatelyEqualRel(double a, double b, double relEpsilon)
{
	return (std::abs(a - b) <= (std::max(std::abs(a), std::abs(b)) * relEpsilon));
}
```

- In this case, instead of epsilon being an absolute number, epsilon is now relative to the magnitude of _a_ or _b_.