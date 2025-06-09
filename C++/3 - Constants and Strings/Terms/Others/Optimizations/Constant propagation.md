---
tags: cpp, compiler, optimization, fundamental
---

- **Constant propagation** is an optimization technique where the compiler replaces variables known to have constant values with their values.

```cpp
#include <iostream>

int main()
{
	int x { 7 };
	std::cout << x << '\n'; // x get replaced by 7 during compilation process here

	return 0;
}
```