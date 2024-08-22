- C++17 introduces the **constexpr if statement**, which requires the conditional to be a constant expression. The conditional of a constexpr-if-statement will be evaluated at compile-time.
- If the constexpr conditional evaluates to `true`, the entire if-else will be replaced by the true-statement. If the constexpr conditional evaluates to `false`, the entire if-else will be replaced by the false-statement (if it exists) or nothing (if there is no else).

- To use a constexpr-if-statement, we add the `constexpr` keyword after the `if`:
```cpp
#include <iostream>

int main()
{
	constexpr double gravity{ 9.8 };

	if constexpr (gravity == 9.8) // now using constexpr if
		std::cout << "Gravity is normal.\n";
	else
		std::cout << "We are not on Earth.\n";

	return 0;
}
```

- When the above code is compiled, the compiler will evaluate the conditional at compile time, see that it is always `true`, and keep only the single statement `std::cout << "Gravity is normal.\n";`.

- In other words, it will compile this:
```cpp
int main()
{
	constexpr double gravity{ 9.8 };

	std::cout << "Gravity is normal.\n";

	return 0;
}
```