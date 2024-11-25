### Context
```cpp
#include <iostream>

void printValue(int y)
{
    std::cout << y << '\n';
} // y is destroyed here

int main()
{
    int x { 2 };

    printValue(x); // x is passed by value (copied) into parameter y (inexpensive)

    return 0;
}
```

- In the above program, when `printValue(x)` is called, the value of `x` (`2`) is _copied_ into parameter `y`. Then, at the end of the function, object `y` is destroyed.
- This means that when we called the function, we made a copy of our argument’s value, only to use it briefly and then destroy it! Fortunately, because fundamental types are cheap to copy, this isn’t a problem. But for large object, like a string, it can be expensive to copy.

### Pass by reference
- One way to avoid making an expensive copy of an argument when calling a function is to use `pass by reference` instead of `pass by value`. When using **pass by reference**, we declare a function parameter as a reference type (or const reference type) rather than as a normal type. When the function is called, each reference parameter is bound to the appropriate argument. Because the reference acts as an alias for the argument, no copy of the argument is made.
- Passing values by reference to non-const allows us to write functions that modify the value of arguments passed in.

### Pass by const lvalue reference
- Unlike a reference to non-const (which can only bind to modifiable lvalues), a reference to const can bind to modifiable lvalues, non-modifiable lvalues, and rvalues.
- Passing by const reference offers the same primary benefit as pass by reference (avoiding making a copy of the argument), while also guaranteeing that the function can _not_ change the value being referenced.

```ad-warning
Passing by const reference offers the same primary benefit as pass by reference (avoiding making a copy of the argument), while also guaranteeing that the function can _not_ change the value being referenced.
```

```ad-note
Favor passing by const reference over passing by non-const reference unless you have a specific reason to do otherwise (e.g. the function needs to change the value of an argument).
```