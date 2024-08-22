## std::string
- The easiest way to work with strings and string objects in C++ is via the `std::string` type, which lives in the \<string\> header.

```cpp
#include <string> // allows use of std::string

int main()
{
    std::string name {}; // empty string

    return 0;
}
```

### Use std::getline() to input text
- To read a full line of input into a string, you’re better off using the `std::getline()` function instead. 
- `std::getline()` requires two arguments: the first is `std::cin`, and the second is your string variable.

```cpp
#include <iostream>
#include <string> // For std::string and std::getline

int main()
{
    std::cout << "Enter your full name: ";
    std::string name{};
    std::getline(std::cin >> std::ws, name); // read a full line of text into name

    std::cout << "Enter your favorite color: ";
    std::string color{};
    std::getline(std::cin >> std::ws, color); // read a full line of text into color

    std::cout << "Your name is " << name << " and your favorite color is " << color << '\n';

    return 0;
}
```

```ad-note
We can use the `std::ws` input manipulator, to tell `std::getline()` to ignore any leading whitespace characters. This is not preserved across calls.
```

### Use std::string::length()
- If we want to know how many characters are in a `std::string`, we can ask a `std::string` object for its length.
```ad-note
Although `std::string` is required to be null-terminated (as of C++11), the returned length of a `std::string` does not include the implicit null-terminator character.
```
- Also note that `std::string::length()` returns an unsigned integral value (most likely of type `size_t`). If you want to assign the length to an `int` variable, you should '[[Type conversion|static_cast]]' it to avoid compiler warnings about signed/unsigned conversions:

```cpp
int length { static_cast<int>(name.length()) };
```

- In C++20, you can also use the `std::ssize()` function to get the length of a `std::string` as a large signed integral type (usually `std::ptrdiff_t`):

```cpp
#include <iostream>
#include <string>

int main()
{
    std::string name{ "Alex" };
    std::cout << name << " has " << std::ssize(name) << " characters\n";

    return 0;
}
```

### Initialing a std::string is expensive
- **Whenever a std::string is initialized, a copy of the string used to initialize it is made.** Making copies of strings is expensive, so care should be taken to minimize the number of copies made.

### Literals for std::string
- Double-quoted string literals (like “Hello, world!”) are C-style strings by default (and thus, have a strange type).
- We can create string literals with type `std::string` by using a `s` suffix after the double-quoted string literal. The `s` must be lower case.
```ad-tip
When you create a `std::string` object using the `s` literal suffix (introduced in C++14), it indeed creates a temporary `std::string` object. If it’s used directly in an expression, its lifetime lasts until the end of the full expression in which it is used.
```

```cpp
#include <iostream>
#include <string> // for std::string

int main()
{
    using namespace std::string_literals; // easy access to the s suffix

    std::cout << "foo\n";   // no suffix is a C-style string literal
    std::cout << "goo\n"s;  // s suffix is a std::string literal

    return 0;
}
```