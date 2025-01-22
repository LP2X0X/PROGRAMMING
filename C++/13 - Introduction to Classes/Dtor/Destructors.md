- Classes have another type of special member function that is called automatically when an object of a non-aggregate class type is destroyed. This function is called a **destructor**. Destructors are designed to allow a class to do any necessary clean up before an object of the class is destroyed.
- Destructors have specific naming rules:
	1. The destructor must have the same name as the class, preceded by a tilde (~).
	2. The destructor can not take arguments.
	3. The destructor has no return type.

```cpp
#include <iostream>

class Simple
{
private:
    int m_id {};

public:
    Simple(int id)
        : m_id { id }
    {
        std::cout << "Constructing Simple " << m_id << '\n';
    }

    ~Simple() // here's our destructor
    {
        std::cout << "Destructing Simple " << m_id << '\n';
    }

    int getID() const { return m_id; }
};

int main()
{
    // Allocate a Simple
    Simple simple1{ 1 };
    {
        Simple simple2{ 2 };
    } // simple2 dies here

    return 0;
} // simple1 dies here
```

```ad-note
Remember that static variables (including global variables and static local variables) are constructed at program startup and destroyed at program shutdown.
```