- Each member of a class type has a property called an **access level** that determines who can access that member.
- C++ has three different access levels: _public_, _private_, and _protected_.
- Whenever a member is accessed, the compiler checks whether the access level of the member permits that member to be accessed. If the access is not permitted, the compiler will generate a compilation error. This access level system is sometimes informally called **access controls**.

---

### The members of a struct are public by default
- Members that have the _public_ access level are called _public members_. **Public members** are members of a class type that do not have any restrictions on how they can be accessed. Much like the park in our opening analogy, public members can be accessed by anyone (as long as they are in scope).
- Public members can be accessed by other members of the same class. Notably, public members can also be accessed by **the public**, which is what we call code that exists _outside_ the members of a given class type. Examples of _the public_ include non-member functions, as well as the members of other class types.

```ad-note
The members of a struct are public by default. Public members can be accessed by other members of the class type, and by the public.

The term “the public” is used to refer to code that exists outside of the members of a given class type. This includes non-member functions, as well as the members of other class types.
```

```ad-important
By default, all members of a struct are public members.
```

---

### The members of a class are private by default

- Members that have the _private_ access level are called _private members_. **Private members** are members of a class type that can only be accessed by other members of the same class.

```ad-note
The members of a class are private by default. Private members can be accessed by other members of the class, but can not be accessed by the public.

A class with private members is no longer an aggregate, and therefore can no longer use aggregate initialization.
```

---

### Naming your private member variables

- In C++, it is a common convention to name private data members starting with an “m_” prefix. This is done for a couple of important reasons.
	- First, the “m_” prefix allows us to easily differentiate data members from function parameters or local variables within a member function. This is the same reason we recommend using “s_” prefixes for local static variables, and “g_” prefixes for globals.
	- Second, the “m_” prefix helps prevent naming collisions between private member variables and the names of local variables, function parameters, and member functions.

---

### Setting access levels via access specifiers
- We can explicitly set the access level of our members by using an **access specifier**. An access specifier sets the access level of _all members_ that follow the specifier. C++ provides three access specifiers: `public:`, `private:`, and `protected:`.

```cpp
class Date
{
// Any members defined here would default to private

public: // here's our public access specifier

    void print() const // public due to above public: specifier
    {
        // members can access other private members
        std::cout << m_year << '/' << m_month << '/' << m_day;
    }

private: // here's our private access specifier

    int m_year { 2020 };  // private due to above private: specifier
    int m_month { 14 }; // private due to above private: specifier
    int m_day { 10 };   // private due to above private: specifier
};

int main()
{
    Date d{};
    d.print();  // okay, main() allowed to access public members

    return 0;
}
```

---

### Access level summary

| Access level | Access specifier | Member access | Derived class access | Public access |
| ------------ | ---------------- | ------------- | -------------------- | ------------- |
| Public       | public:          | yes           | yes                  | yes           |
| Protected    | protected:       | yes           | yes                  | no            |
| Private      | private:         | yes           | no                   | no            |

---

### Access levels work on a per-class basis
- You already know that a member function can directly access private members (of the implicit object). However, because access levels are per-class, not per-object, a member function can also directly access the private members of ANY other object of the same class type that is in scope.

```cpp
#include <iostream>
#include <string>
#include <string_view>

class Person
{
private:
    std::string m_name{};

public:
    void kisses(const Person& p) const
    {
        std::cout << m_name << " kisses " << p.m_name << '\n';
    }

    void setName(std::string_view name)
    {
        m_name = name;
    }
};

int main()
{
    Person joe;
    joe.setName("Joe");

    Person kate;
    kate.setName("Kate");

    joe.kisses(kate);

    return 0;
}
```

- Because `kisses()` is a member function, it has direct access to private member `m_name`. However, you might be surprised to see that it also has direct access to `p.m_name`! This works because `p` is a `Person` object, and `kisses()` can access the private members of any `Person` object in scope!