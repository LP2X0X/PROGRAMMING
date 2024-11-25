```cpp
#include <iostream>

template <typename T, typename U>
auto max(T x, U y)
{
    return (x < y) ? y : x;
}

int main()
{
    std::cout << max(2, 3.5) << '\n';

    return 0;
}
```
- If we need a function that can be forward declared, we have to be explicit about the return type. Since our return type needs to be the common type of `T` and `U`, we can use `std::common_type_t` to fetch the common type of `T` and `U` to use as our explicit return type:

```cpp
#include <iostream>

template <typename T, typename U>
auto max(T x, U y) -> std::common_type_t<T, U>; // returns the common type of T and U

int main()
{
    std::cout << max(2, 3.5) << '\n';

    return 0;
}

template <typename T, typename U>
auto max(T x, U y) -> std::common_type_t<T, U>
{
    return (x < y) ? y : x;
}
```