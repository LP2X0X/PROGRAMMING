A **struct** in C++ is a user-defined data type that groups variables (of potentially different types) under one name. It's similar to a class, with the key difference being that members of a `struct` are **public by default**, while members of a `class` are **private by default**.

Structs are widely used to define lightweight objects or simple data containers.

---

### **Syntax**

Here’s the basic syntax for defining a `struct`:

```cpp
struct StructName
{
    // Member variables (fields)
    int x;
    float y;

    // Member functions (optional)
    void display()
    {
        std::cout << "x: " << x << ", y: " << y << std::endl;
    }
};
```

---

### **Key Features**

1. **Public by Default**: All members in a `struct` are public unless explicitly specified otherwise (with `private` or `protected` access modifiers).
2. **Supports Member Functions**: Like a `class`, a `struct` can contain member functions.
3. **Inheritance**: Structs can inherit from other classes or structs.
4. **Templates**: Structs can be templated.
5. **Constructors and Destructors**: Structs can have constructors and destructors.
6. **Friend Functions**: Structs can have friend functions or classes.

---

### **Example**

#### 1. **Simple Struct Example**

```cpp
#include <iostream>
#include <string>

struct Point
{
    int x; // Public member
    int y; // Public member

    void display() // Member function
    {
        std::cout << "Point(" << x << ", " << y << ")" << std::endl;
    }
};

int main()
{
    Point p1 {10, 20}; // Initialize struct
    p1.display();      // Access member function

    return 0;
}
```

Output:

```
Point(10, 20)
```

---

#### 2. **Using Access Modifiers**

```cpp
#include <iostream>

struct Rectangle
{
private:
    int length; // Private member
    int width;  // Private member

public:
    // Constructor
    Rectangle(int l, int w) : length(l), width(w) {}

    // Public function to calculate area
    int area() const
    {
        return length * width;
    }

    // Setter for length
    void setLength(int l)
    {
        if (l > 0)
            length = l;
    }

    // Getter for length
    int getLength() const
    {
        return length;
    }
};

int main()
{
    Rectangle rect(10, 5);
    std::cout << "Area: " << rect.area() << std::endl;

    rect.setLength(15); // Modify private member using setter
    std::cout << "New Length: " << rect.getLength() << std::endl;

    return 0;
}
```

Output:

```
Area: 50
New Length: 15
```

---

#### 3. **Struct with Inheritance**

```cpp
#include <iostream>

struct Base
{
    void showBase() const
    {
        std::cout << "This is the Base struct" << std::endl;
    }
};

struct Derived : Base
{
    void showDerived() const
    {
        std::cout << "This is the Derived struct" << std::endl;
    }
};

int main()
{
    Derived d;
    d.showBase();    // Call base class function
    d.showDerived(); // Call derived class function

    return 0;
}
```

Output:

```
This is the Base struct
This is the Derived struct
```

---

### **Struct vs Class**

|Feature|`struct`|`class`|
|---|---|---|
|**Default Access**|Public|Private|
|**Use Case**|Typically for simple data structures|Used for more complex behavior|
|**Inheritance**|Public inheritance by default|Private inheritance by default|

---

### **Advanced Example: Templated Struct**

```cpp
#include <iostream>

template <typename T>
struct Box
{
    T value;

    void display() const
    {
        std::cout << "Value: " << value << std::endl;
    }
};

int main()
{
    Box<int> intBox {42};
    intBox.display();

    Box<std::string> strBox {"Hello"};
    strBox.display();

    return 0;
}
```

Output:

```
Value: 42
Value: Hello
```

---

### When to Use `struct`?

- Use `struct` for **lightweight data types** with public access.
- Use `class` when you need more complex encapsulation, private/protected members by default, or when it represents a concept rather than just a collection of data.