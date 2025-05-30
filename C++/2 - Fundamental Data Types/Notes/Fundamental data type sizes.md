---
tags: cpp, note, fundamental
---

- The C++ standard does not define the exact size (in bits) of any of the fundamental types. Instead, the standard says the following:
	- An object must occupy at least 1 byte (so that each object has a distinct memory address).
	- A byte must be at least 8 bits.
	- The integral types `char`, `short`, `int`, `long`, and `long long` have a minimum size of 8, 16, 16, 32, and 64 bits respectively.
	- `char` and `char8_t` are exactly 1 byte (at least 8 bits).