---
tags:
  - cpp
  - compiler
  - optimization
  - fundamental
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

---

### Const variables are easier to optimize

- Since `x` is defined as a non-const variable, in order to apply this optimization, the compiler must realize that the value of `x` actually doesn’t change (even though it could). Whether the compiler is capable of doing so comes down to how complex the program is and how sophisticated the compiler’s optimization routines are.

- We can help the compiler optimize more effectively by using constant variables wherever possible. For example:

```cpp
#include <iostream>

int main()
{
	const int x { 7 }; // x is now const
	std::cout << x << '\n';

	return 0;
}
```

- Because `x` is now const, the compiler has a guarantee that `x` can’t be changed after initialization. This makes it more likely the compiler will apply constant propagation, and then optimize the variable out entirely.