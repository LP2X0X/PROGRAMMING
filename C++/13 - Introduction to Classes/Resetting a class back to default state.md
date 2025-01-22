If your class has a default constructor, you may be interested in providing a way to return an existing object back to its default state.

The best way to reset a class back to a default state is to create a `reset()` member function, have that function create a new object (using the default constructor), and then assign that new object to the current implicit object, like this:

```cpp
void reset()
{
    *this = {}; // value initialize a new object and overwrite the implicit object
}
```

Here’s a full program demonstrating this `reset()` function in action:

```cpp
#include <iostream>

class Calc
{
private:
    int m_value{};

public:
    Calc& add(int value) { m_value += value; return *this; }
    Calc& sub(int value) { m_value -= value; return *this; }
    Calc& mult(int value) { m_value *= value; return *this; }

    int getValue() const { return m_value; }

    void reset() { *this = {}; } // reset function here
};


int main()
{
    Calc calc{};
    calc.add(5).sub(3).mult(4);

    std::cout << calc.getValue() << '\n'; // prints 8

    calc.reset();

    std::cout << calc.getValue() << '\n'; // prints 0

    return 0;
}
```