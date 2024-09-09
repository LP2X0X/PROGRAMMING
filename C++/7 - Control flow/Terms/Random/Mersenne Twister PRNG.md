- The Mersenne Twister PRNG, besides having a great name, is probably the most popular PRNG across all programming languages. Although it is a bit old by today’s standards, it generally produces quality results and has decent performance. The random library has support for two Mersenne Twister types:
	- `mt19937` is a Mersenne Twister that generates 32-bit unsigned integers
	- `mt19937_64` is a Mersenne Twister that generates 64-bit unsigned integers
</br>
- Using Mersenne Twister is straightforward:

```cpp
#include <iostream>
#include <random> // for std::mt19937

int main()
{
	std::mt19937 mt{}; // Instantiate a 32-bit Mersenne Twister

	// Print a bunch of random numbers
	for (int count{ 1 }; count <= 40; ++count)
	{
		std::cout << mt() << '\t'; // generate a random number

		// If we've printed 5 numbers, start a new row
		if (count % 5 == 0)
			std::cout << '\n';
	}

	return 0;
}
```