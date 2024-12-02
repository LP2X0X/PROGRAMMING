- Functions that belong to a class type are called **member functions**.
- Functions that are not member functions are called **non-member functions** (or occasionally **free functions**) to distinguish them from member functions.

---

### Returning data members by lvalue reference
- Data members have the same lifetime as the object containing them. Since member functions are always called on an object, and that object must exist in the scope of the caller, it is generally safe for a member function to return a data member by (const) lvalue reference (as the member being returned by reference will still exist in the scope of the caller when the function returns).

```cpp
#include <iostream>
#include <string>

class Employee
{
	std::string m_name{};

public:
	void setName(std::string_view name) { m_name = name; }
	const std::string& getName() const { return m_name; } //  getter returns by const reference
};

int main()
{
	Employee joe{}; // joe exists until end of function
	joe.setName("Joe");

	std::cout << joe.getName(); // returns joe.m_name by reference

	return 0;
}
```

```ad-tip
A member function returning a reference should return a reference of the same type as the data member being returned, to avoid unnecessary conversions.
```

- In general, the return type of a member function returning by reference should match the type of the data member being returned. In the above example, `m_name` is of type `std::string`, so `getName()` returns `const std::string&`.
- Returning a `std::string_view` would require a temporary `std::string_view` to be created and returned every time the function was called. That’s needlessly inefficient. If the caller wants a `std::string_view`, they can do the conversion themselves.
