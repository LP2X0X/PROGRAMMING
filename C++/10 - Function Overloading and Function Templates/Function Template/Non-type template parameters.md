- A **non-type template parameter** is a template parameter with a fixed type that serves as a placeholder for a constexpr value passed in as a template argument.
- A non-type template parameter can be any of the following types:
	- An integral type
	- An enumeration type
	- `std::nullptr_t`
	- A floating point type (since C++20)
	- A pointer or reference to an object
	- A pointer or reference to a function
	- A pointer or reference to a member function
	- A literal class type (since C++20)
- Non-type template parameters are used primarily when we need to pass constexpr values to functions (or class types) so they can be used in contexts that require a constant expression.

---

### Type deduction for non-type template parameters using `auto` C++17
- Unlike with normal functions, the algorithm for matching function template calls to function template definitions is not sophisticated, and certain matches are not prioritized over others based on the type of conversion required (or lack thereof). This means that if a function template is overloaded for different kinds of non-type template parameters, it can very easily result in an ambiguous match.
- As of C++17, non-type template parameters may use `auto` to have the compiler deduce the non-type template parameter from the template argument:

```cpp
#include <iostream>

template <auto N> // deduce non-type template parameter from template argument
void print()
{
    std::cout << N << '\n';
}

int main()
{
    print<5>();   // N deduced as int `5`
    print<'c'>(); // N deduced as char `c`

    return 0;
}
```