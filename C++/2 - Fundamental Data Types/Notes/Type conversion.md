---
tags:
  - cpp
  - note
  - fundamental
---

- The process of converting a value from one type to another type is called **type conversion**.

---

### Implicit type conversion

- When the compiler does type conversion on our behalf without us explicitly asking, we call this **implicit type conversion**.
```cpp
#include <iostream>

void print(double x) // print takes a double parameter
{
	std::cout << x << '\n';
}

int main()
{
	print(5); // what happens when we pass an int value?
			  // it will be implicitly convert to double

	return 0;
}
```
---

### Explicit type conversion

- **Explicit type conversion** allow us (the programmer) to explicitly tell the compiler to convert a value from one type to another type, and that we take full responsibility for the result of that conversion. If such a conversion results in the loss of value, the compiler will not warn us.
- To perform an explicit type conversion, in most cases we’ll use the `static_cast` operator. The syntax for the `static cast` looks a little funny:
```cpp
static_cast<new_type>(expression)
```

```ad-note
It’s worth noting that the argument to _static_cast_ evaluates as an [[Expression|expression]]. When we pass in a variable, that variable is evaluated to produce its value, and that value is then converted to the new type. The variable itself is _not_ affected by casting its value to a new type.
```

- We can use _static_cast<>_ to convert an integer to a floating point number so that we can do [[Integer and floating point division|floating point division instead of integer division]]:
```cpp
#include <iostream>

int main()
{
    constexpr int x{ 7 };
    constexpr int y{ 4 };

    std::cout << "int / int = " << x / y << '\n';
    std::cout << "double / int = " << static_cast<double>(x) / y << '\n';
    std::cout << "int / double = " << x / static_cast<double>(y) << '\n';
    std::cout << "double / double = " << static_cast<double>(x) / static_cast<double>(y) << '\n';

    return 0;
}
```

This produces the result:

```
int / int = 1
double / int = 1.75
int / double = 1.75
double / double = 1.75
```
