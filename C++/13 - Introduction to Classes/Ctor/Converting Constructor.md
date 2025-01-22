- A constructor that can be used to perform an implicit conversion is called a **converting constructor**. By default, all constructors are converting constructors.

---

### Only one user-defined conversion may be applied

```cpp
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
private:
    std::string m_name{};

public:
    Employee(std::string_view name)
        : m_name{ name }
    {
    }

    const std::string& getName() const { return m_name; }
};

void printEmployee(Employee e) // has an Employee parameter
{
    std::cout << e.getName();
}

int main()
{
    using namespace std::literals;
    printEmployee( "Joe"sv); // now a std::string_view literal
    return 0;
}
```