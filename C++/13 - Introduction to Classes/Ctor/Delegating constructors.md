- Initialization of a class object is finished once the member initializer list has finished executing. By the time we begin executing the constructor’s body, it’s too late to do more initialization.

```ad-note
Constructors should not be called directly from the body of another function. Doing so will either result in a compilation error, or will direct-initialize a temporary object.  
If you do want a temporary object, prefer list-initialization (which makes it clear you are intending to create an object).
```

---

### Delegating constructors

- Constructors are allowed to delegate (transfer responsibility for) initialization to another constructor from the same class type. This process is sometimes called **constructor chaining** and such constructors are called **delegating constructors**.
- To make one constructor delegate initialization to another constructor, simply call the constructor in the member initializer list:

```cpp
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
private:
    std::string m_name { "???" };
    int m_id { 0 };

public:
    Employee(std::string_view name)
        : Employee{ name, 0 } // delegate initialization to Employee(std::string_view, int) constructor
    {
    }

    Employee(std::string_view name, int id)
        : m_name{ name }, m_id { id } // actually initializes the members
    {
        std::cout << "Employee " << m_name << " created\n";
    }

};

int main()
{
    Employee e1{ "James" };
    Employee e2{ "Dave", 42 };
}
```

```ad-important
A few additional notes about delegating constructors. First, a constructor that delegates to another constructor is not allowed to do any member initialization itself. So your constructors can delegate or initialize, but not both.
```