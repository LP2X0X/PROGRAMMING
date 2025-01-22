```ad-important
Prefer using the member initializer list to initialize your members over assigning values in the body of the constructor.
```

```cpp
#include <iostream>

class Foo
{
private:
    int m_x {};
    int m_y {};

public:
    Foo(int x, int y)
        : m_x { x }, m_y { y } // here's our member initialization list
    {
        std::cout << "Foo(" << x << ", " << y << ") constructed\n";
    }

    void print() const
    {
        std::cout << "Foo(" << m_x << ", " << m_y << ")\n";
    }
};

int main()
{
    Foo foo{ 6, 7 };
    foo.print();

    return 0;
}
```

- The **member initializer list** is defined after the constructor parameters. It begins with a colon (:), and then lists each member to initialize along with the initialization value for that variable, separated by a comma.

### Format
- Put the colon on the line after the constructor name, as this cleanly separates the member initializer list from the function prototype.
- Indent your member initializer list, to make it easier to see the function names.
- Because the C++ standard says so, the members in a member initializer list are always initialized in the order in which they are defined inside the class (not in the order they are defined in the member initializer list).

```ad-important
Member variables in a member initializer list should be listed in order that they are defined in the class or else it might cause error.
```