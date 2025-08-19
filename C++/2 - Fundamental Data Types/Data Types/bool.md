---
tags:
  - cpp
  - datatype
  - fundamental
---

- **Boolean** (or *bool* for short) variables are variables that can have only two possible values: `true`, and `false`.
  
```ad-note
Boolean values are not actually stored in Boolean variables as the words “true” or “false”. Instead, they are stored as [[Integer|integral]] values: `true` is stored as integer `1`, and `false` is stored as integer `0`.
Because Booleans store integral values, they are considered to be an *integral type*.
```

```ad-tip
When using [[List-initialization|uniform initialization]], you can initialize a variable using integer literals `0` (for `false`) and `1` (for `true`) (but you really should be using `false` and `true` instead).
```

```ad-warning
Initialize bool variable using list initialization with integer literals can cause compilation errors if its not 0 or 1 since it does not allow narrow conversion.
However, other type of initialization works well.
```

---

### Printing bool values

- When we print Boolean values, `std::cout` prints `0` for `false`, and `1` for `true`:

```cpp
#include <iostream>

int main()
{
    std::cout << true << '\n'; // true evaluates to 1
    std::cout << !true << '\n'; // !true evaluates to 0

    bool b {false};
    std::cout << b << '\n'; // b is false, which evaluates to 0
    std::cout << !b << '\n'; // !b is true, which evaluates to 1
    return 0;
}
```

- Outputs:

```
1
0
0
1
```

- If you want `std::cout` to print `true` or `false` instead of `0` or `1`, you can output `std::boolalpha`. This doesn’t output anything, but manipulates the way `std::cout` outputs bool values.
- You can use `std::noboolalpha` to turn it back off.

```ad-note
Copy initialization allows implicit conversion from int to bool
```

---

### Input bool values
- By default, `std::cin` only accepts numeric input for Boolean variables: `0` is `false`, and `1` is `true`. Any other numeric value will be interpreted as `true`, and will cause `std::cin` to enter failure mode. Any non-numeric value will be interpreted as `false` and will cause `std::cin` to enter failure mode.

- To allow `std::cin` to accept `false` and `true` as inputs, you must first input to `std::boolalpha`. However, when `std::boolalpha` is enabled for input, `0` and `1` will no longer be interpreted as Booleans inputs.

```ad-note
This does not affect std::cout.
```

```cpp
#include <iostream>

int main()
{
	bool b{};
	std::cout << "Enter a boolean value: ";

	// Allow the user to input 'true' or 'false' for boolean values
	// This is case-sensitive, so True or TRUE will not work
	std::cin >> std::boolalpha;
	std::cin >> b;

	// Let's also output bool values as `true` or `false`
	std::cout << std::boolalpha;
	std::cout << "You entered: " << b << '\n';

	return 0;
}
```

```ad-tip
Boolean values are often used as the return values for functions that check whether something is true or not. Such functions are typically named starting with the word _is_ (e.g. isEqual) or _has_ (e.g. hasCommonDivisor).
```