- Member functions can be made static as well. Here is the above example with a static member function accessor:

```cpp
#include <iostream>

class Something
{
private:
    static inline int s_value { 1 };

public:
    static int getValue() { return s_value; } // static member function
};

int main()
{
    std::cout << Something::getValue() << '\n';
}
```

- Because static member functions are not associated with a particular object, they can be called directly by using the class name and the scope resolution operator (e.g. `Something::getValue()`). Like static member variables, they can also be called through objects of the class type, though this is not recommended.

---

### Static member functions have no `this` pointer

- Static member functions have two interesting quirks worth noting. First, because static member functions are not attached to an object, they have no `this` pointer! This makes sense when you think about it -- the `this` pointer always points to the object that the member function is working on. Static member functions do not work on an object, so the `this` pointer is not needed.

- Second, static member functions can directly access other static members (variables or functions), but not non-static members. This is because non-static members must belong to a class object, and static member functions have no class object to work with!

---

### Static members defined outside the class definition

- Static member functions can also be defined outside of the class declaration. This works the same way as for normal member functions.

```cpp
#include <iostream>

class IDGenerator
{
private:
    static inline int s_nextID { 1 };

public:
     static int getNextID(); // Here's the declaration for a static function
};

// Here's the definition of the static function outside of the class.  Note we don't use the static keyword here.
int IDGenerator::getNextID() { return s_nextID++; }

int main()
{
    for (int count{ 0 }; count < 5; ++count)
        std::cout << "The next ID is: " << IDGenerator::getNextID() << '\n';

    return 0;
}
```

---

### Classes with all static members
- Although such “pure static classes” (also called “monostates”) can be useful, they also come with some potential downsides:
	- First, because all static members are instantiated only once, there is no way to have multiple copies of a pure static class (without cloning the class and renaming it).
	- Because all of the members belong to the class (instead of object of the class), and class declarations usually have global scope, a pure static class is essentially the equivalent of declaring functions and global variables in a globally accessible namespace, with all the requisite downsides that global variables have.

```ad-tip
Instead of writing a class with all static members, consider writing a normal class and instantiating a global instance of it (global variables have static duration). That way the global instance can be used when appropriate, but local instances can still be instantiated if and when that is useful.
```

---

### C++ does not support static constructors
- If your static variable can be directly initialized, no constructor is needed: you can initialize the static member variable at the point of definition (even if it is private)``.
- If initializing your static member variable requires executing code (e.g. a loop), there are many different, somewhat obtuse ways of doing this. One way that works with all variables, static or not, is to use a function to create an object, fill it with data, and return it to the caller. This returned value can be copied into the object being initialized.