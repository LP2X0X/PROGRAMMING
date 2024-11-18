The `auto` keyword can also be used to declare functions using a **trailing return syntax**, where the return type is specified after the rest of the function prototype.

Consider the following function:

```cpp
int add(int x, int y)
{
  return (x + y);
}
```

Using the trailing return syntax, this could be equivalently written as:

```cpp
auto add(int x, int y) -> int
{
  return (x + y);
}
```

In this case, `auto` does not perform type deduction -- it is just part of the syntax to use a trailing return type.

Why would you want to use this? Here are some reasons:

1. For functions with complex return types, a trailing return type can make the function easier to read:

```cpp
#include <type_traits> // for std::common_type

std::common_type_t<int, double> compare(int, double);         // harder to read (where is the name of the function in this mess?)
auto compare(int, double) -> std::common_type_t<int, double>; // easier to read (we don't have to read the return type unless we care)
```

2. The trailing return type syntax can be used to align the names of your functions, which makes consecutive function declarations easier to read:

```cpp
auto add(int x, int y) -> int;
auto divide(double x, double y) -> double;
auto printSomething() -> void;
auto generateSubstring(const std::string &s, int start, int len) -> std::string;
```

3. If we have a function whose return type must be deduced based on the type of the function parameters, a normal return type will not suffice, because the compiler has not yet seen the parameters at that point.

```cpp
std::common_type_t<x, y> add(int x, double y);         // Compile error: compiler hasn't seen definitions of x and y yet
auto add(int x, double y) -> std::common_type_t<x, y>; // ok
```

Because the compiler can determine the return type from the function declaration, such functions do not need to be defined before being used.

4. The trailing return syntax is also required for some advanced features of C++, such as lambdas.

For now, we recommend the continued use of the traditional function return syntax except in situations that require the trailing return syntax.