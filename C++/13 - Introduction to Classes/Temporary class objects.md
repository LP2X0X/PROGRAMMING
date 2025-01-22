- A **temporary object** (sometimes called an **anonymous object** or an **unnamed object**) is an object that has no name and exists only for the duration of a single expression.

```cpp
#include <iostream>

class IntPair
{
private:
    int m_x{};
    int m_y{};

public:
    IntPair(int x, int y)
        : m_x { x }, m_y { y }
    {}

    int x() const { return m_x; }
    int y() const{ return m_y; }
};

void print(IntPair p)
{
    std::cout << "(" << p.x() << ", " << p.y() << ")\n";
}

int main()
{
    // Case 1: Pass variable
    IntPair p { 3, 4 };
    print(p);

    // Case 2: Construct temporary IntPair and pass to function
    print(IntPair { 5, 6 } );

    // Case 3: Implicitly convert { 7, 8 } to a temporary Intpair and pass to function
    print( { 7, 8 } );

    return 0;
}
```

- In case 2, we’re telling the compiler to construct an `IntPair` object, and initializing it with `{ 5, 6 }`. Because this object has no name, it is a temporary. The temporary object is then passed to parameter `p` of function `print()`. When the function call returns, the temporary object is destroyed.

- In case 3, we’re also creating a temporary `IntPair` object to pass to function `print()`. However, because we have not explicitly specified what type to construct, the compiler will deduce the necessary type (`IntPair`) from the function parameter, and then implicitly convert `{ 7, 8 }` to an `IntPair` object.

---

```ad-note
As a quick rule of thumb: Prefer `static_cast` when converting to a fundamental type, and a list-initialized temporary when converting to a class type.

Prefer `static_cast` when to create a temporary object when any of the following are true:

- We need to performing a narrowing conversion.
- We want to make it really obvious that we’re converting to a type that will result in some different behavior (e.g. a `char` to an `int`).
- We want to use direct-initialization for some reason (e.g. to avoid list constructors taking precedence).

Prefer creating a new object (using list initialization) to create a temporary object when any of the following are true:

- We want to use list-initialization (e.g. for the protection against narrowing conversions, or because we need to invoke a list constructor).
- We need to provide additional arguments to a constructor to facilitate the conversion.
```