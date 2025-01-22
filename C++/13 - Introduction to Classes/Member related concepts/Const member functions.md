- A **const member function** is a member function that guarantees it will not modify the object or call any non-const member functions (as they may modify the object).
- Turning a member function to a const member function is easy -- we simply append the `const` keyword to the function prototype, after the parameter list, but before the function body:

```cpp
#include <iostream>

struct Date
{
    int year {};
    int month {};
    int day {};

    void print() const // now a const member function
    {
        std::cout << year << '/' << month << '/' << day;
    }
};

int main()
{
    const Date today { 2020, 10, 14 }; // const

    today.print();  // ok: const object can call const member function

    return 0;
}
```

```ad-tip
A member function that does not (and will not ever) modify the state of the object should be made const, so that it can be called on both const and non-const objects.
```