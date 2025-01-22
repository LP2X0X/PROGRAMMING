- Not allowing class types with private members to be initialized via aggregate initialization makes sense for a number of reasons:
	- Aggregate initialization requires knowing about the implementation of the class (since you have to know what the members are, and what order they were defined in), which we’re intentionally trying to avoid when we hide our data members.
	- If our class had some kind of invariant, we’d be relying on the user to initialize the class in a way that preserves the invariant.

---

### Definition
- A **constructor** is a special member function that is automatically called after a non-aggregate class type object is created.
- When a non-aggregate class type object is defined, the compiler looks to see if it can find an accessible constructor that is a match for the initialization values provided by the caller (if any).
	- If an accessible matching constructor is found, memory for the object is allocated, and then the constructor function is called.
	- If no accessible matching constructor can be found, a compilation error will be generated.

```ad-note
Many new programmers are confused about whether constructors create the objects or not. They do not -- the compiler sets up the memory allocation for the object prior to the constructor call. The constructor is then called on the uninitialized object.

However, if a matching constructor cannot be found for a set of initializers, the compiler will error. So while constructors don’t create objects, the lack of a matching constructor will prevent creation of an object.
```

- Beyond determining how an object may be created, constructors generally perform two functions:
	- They typically perform initialization of any member variables (via a member initialization list)
	- They may perform other setup functions (via statements in the body of the constructor). This might include things such as error checking the initialization values, opening a file or database, etc…

```ad-note
Note that aggregates are not allowed to have constructors -- so if you add a constructor to an aggregate, it is no longer an aggregate.
```

---

### Naming constructors

- Unlike normal member functions, constructors have specific rules for how they must be named:
	- Constructors must have the same name as the class (with the same capitalization). For template classes, this name excludes the template parameters.
	- Constructors have no return type (not even `void`).
- Because constructors are typically part of the interface for your class, they are usually public.

---

### Overloaded constructors

- Because constructors are functions, they can be overloaded. That is, we can have multiple constructors so that we can construct objects in different ways:

```cpp
#include <iostream>

class Foo
{
private:
    int m_x {};
    int m_y {};

public:
    Foo() // default constructor
    {
        std::cout << "Foo constructed\n";
    }

    Foo(int x, int y) // non-default constructor
        : m_x { x }, m_y { y }
    {
        std::cout << "Foo(" << m_x << ", " << m_y << ") constructed\n";
    }
};

int main()
{
    Foo foo1{};     // Calls Foo() constructor
    Foo foo2{6, 7}; // Calls Foo(int, int) constructor

    return 0;
}
```