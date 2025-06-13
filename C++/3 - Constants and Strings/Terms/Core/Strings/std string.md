---
tags: cpp, term, fundamental
---

- The easiest way to work with strings and string objects in C++ is via the `std::string` type, which lives in the \<string> header.
- A `std::string` object can store strings of different lengths.

```ad-note
C-style string literal is always [[null-terminated string|null-terminated]].
```

```cpp
#include <string> // allows use of std::string

int main()
{
    std::string name {}; // empty string
    std::string name { "Alex" }; // initialize name with string literal "Alex"
    name = "John";              // change name to "John"

    return 0;
}
```

---

### Use `std::getline()` to input text

- To read a full line of input into a string, you’re better off using the `std::getline()` function instead. `std::getline()` requires two arguments: the first is `std::cin`, and the second is your string variable. The `std::ws`[[Input manipulator|input manipulator]] tells `std::cin` to ignore any leading whitespace before extraction.

```ad-note
`std::getline()` does not ignore leading whitespace. It stops extracting when encountering a newline.
```

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

---

### The length of a std::string

- If we want to know how many characters are in a `std::string`, we can ask a `std::string` object for its length:

```cpp
#include <iostream>
#include <string>

int main()
{
    std::string name{ "Alex" };
    std::cout << name << " has " << name.length() << " characters\n";

    return 0;
}
```

---

### std::string is expensive
- Whenever a std::string is initialized, a copy of the string used to initialize it is made. Making copies of strings is expensive, so care should be taken to minimize the number of copies made.
- When a `std::string` is passed to a function by value, the `std::string` function parameter must be instantiated and initialized with the argument. This results in an expensive copy therefore we should not do this.
- As a rule of thumb, it is okay to return a `std::string` by value when the expression of the return statement resolves to any of the following:
	- A local variable of type `std::string`.
	```cpp
	std::string getName() {
	    std::string name = "Alice";
	    return name; // Move or RVO happens here
	}
	```
	- A `std::string` that has been returned by value from another function call or operator.
	```cpp
	std::string getFirstName() {
		return "Bob";
	}
	
	std::string getFullName() {
		return getFirstName(); // Efficient: compiler moves or elides
	}
	```
	- A `std::string` temporary that is created as part of the return statement.
	```cpp
	std::string greet() {
	    return std::string("Hello, world!"); // Temporary: moved or elided
	}
	```
- 

