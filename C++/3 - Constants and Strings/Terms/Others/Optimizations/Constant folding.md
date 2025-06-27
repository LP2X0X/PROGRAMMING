---
tags: cpp, compiler, optimization, fundamental
---

- **Constant folding** is an optimization technique where the compiler replaces [[C++/1 - Basic C++/Terms/Expressions/Expression|expressions]] or [[Sub Expression|sub expressions]] that have [[Literals|literal]] operands with the result of the expression.

```cpp
#include <iostream>

int main()
{
	int x { 3 + 4 }; // This is equivalent to: int x { 7 };
	std::cout << x << '\n';

	return 0;
}
```