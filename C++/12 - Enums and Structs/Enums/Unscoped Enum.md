### **Unscoped Enumerations (Classic Enums) in C++**

Unscoped enumerations, commonly referred to as "classic enums," are a feature of C++ that allows you to define a set of named integral constants. They are defined using the `enum` keyword and have been part of C++ since its inception.

```ad-important
**Unscoped enumerations** are named such because they **put their enumerator names into the same scope as the enumeration definition itself** (as opposed to creating a new scope region like a namespace does).
```

---

### **Syntax**

When we define an enumeration, each enumerator is automatically associated with an integer value based on its position in the enumerator list. By default, the first enumerator is given the integral value `0`, and each subsequent enumerator has a value one greater than the previous enumerator.

```ad-note
It is possible to explicitly define the value of enumerators. These integral values can be positive or negative, and can share the same value as other enumerators. Any non-defined enumerators are given a value one greater than the previous enumerator.
```

```ad-warning
If an enumeration is zero-initialized (which happens when we use value-initialization), the enumeration will be given value `0`, even if there is no corresponding enumerator with that value.
```

```cpp
enum Color {
    Red,    // 0
    Green,  // 1
    Blue    // 2
};
```

---

### **Key Characteristics of Unscoped Enums**

1. **Implicit Conversion to Integer**:
    
    - Unscoped enums automatically convert to their underlying integral type (default is `int`).
    - Example:
        
        ```cpp
        Color color = Red;
        int value = color;  // Implicitly converts `color` to 0
        ```
        
2. **Global Scope**:
    
    - Enum constants are placed directly into the enclosing scope.
    - Example:
        
        ```cpp
        Color color = Red;  // No need to qualify with `Color::Red`
        ```
        
3. **Underlying Type**:
    
    - By default, the underlying type is `int`.
    - Can be explicitly specified (since C++11):
        
        ```cpp
        enum Color : unsigned char {
            Red, Green, Blue
        };
        ```
        
4. **Limited Type Safety**:
    
    - Because of implicit conversion to `int`, unscoped enums can accidentally interact with integers or other enums, potentially causing issues.
    - Example:
        
        ```cpp
        Color color = Red;
        color = 10;  // Allowed but may cause logical errors
        ```
        
5. **No Namespace Pollution Prevention**:
    
    - All constants in an unscoped enum share the same enclosing scope, so naming conflicts can occur.
    - Example:
        
        ```cpp
        enum Color { Red, Green, Blue };
        enum TrafficLight { Red, Yellow, Green };  // Naming conflict with `Red` and `Green`
        ```
        

6. **Unscoped enumerations will implicitly convert to integral values**
   
	   - An unscoped enumeration will implicitly convert to an integral value. Because enumerators are compile-time constants, this is a constexpr conversion.
---

### **Example of Unscoped Enum in Use**

```cpp
#include <iostream>

enum Color {
    Red,    // 0
    Green,  // 1
    Blue    // 2
};

int main() {
    Color color = Red;
    std::cout << "Color value: " << color << std::endl;  // Output: 0

    // Implicit integer conversion
    int value = Green;
    std::cout << "Integer value: " << value << std::endl;  // Output: 1

    return 0;
}
```

---

### **Common Use Cases**

- Defining fixed sets of integral constants where strict type safety is not a primary concern.
- Simplifying constant names in small scopes.

---

### **Limitations**

1. **Lack of Type Safety**:
    
    - Unscoped enums allow implicit conversion to integers, which can lead to errors when unintended.
    - Example:
        
        ```cpp
        enum Color { Red, Green, Blue };
        Color color = Green;
        color = 5;  // This compiles but is logically incorrect
        ```
        
2. **Namespace Pollution**:
    
    - Enum constants are part of the enclosing scope, leading to potential naming conflicts.

---

### **Scoped Enum vs. Unscoped Enum**

To overcome the limitations of unscoped enums, **scoped enumerations** were introduced in C++11 using the `enum class` syntax.

|Feature|Unscoped Enum (`enum`)|Scoped Enum (`enum class`)|
|---|---|---|
|**Implicit Conversion**|Converts to integers automatically|Does not implicitly convert to integers|
|**Enclosing Scope**|Enum constants are in the enclosing scope|Enum constants are in the enum's scope|
|**Type Safety**|Limited (implicit conversions allowed)|Strongly typed (no accidental conversion)|
|**Usage Syntax**|`Red`|`Color::Red`|
|**Underlying Type**|`int` (default), customizable since C++11|Customizable, must be explicitly qualified|

---

### **When to Use Unscoped Enums**

- When backward compatibility with older C++ codebases is necessary.
- For small, non-conflicting sets of constants in local or single-use scopes.
- When simplicity and implicit integer conversion are desirable.

For most modern applications, however, **scoped enums (`enum class`)** are recommended due to their type safety and reduced scope pollution.

---

### **Avoiding enumerator naming collisions**

There are quite a few common ways to prevent unscoped enumerator naming collisions.

One option is to prefix each enumerator with the name of the enumeration itself:

```cpp
enum Color
{
    color_red,
    color_blue,
    color_green,
};

enum Feeling
{
    feeling_happy,
    feeling_tired,
    feeling_blue, // no longer has a naming collision with color_blue
};

int main()
{
    Color paint { color_blue };
    Feeling me { feeling_blue };

    return 0;
}
```

This still pollutes the namespace but reduces the chance for naming collisions by making the names longer and more unique.

A better option is to put the enumerated type inside something that provides a separate scope region, such as a namespace:

```cpp
namespace Color
{
    // The names Color, red, blue, and green are defined inside namespace Color
    enum Color
    {
        red,
        green,
        blue,
    };
}

namespace Feeling
{
    enum Feeling
    {
        happy,
        tired,
        blue, // Feeling::blue doesn't collide with Color::blue
    };
}

int main()
{
    Color::Color paint{ Color::blue };
    Feeling::Feeling me{ Feeling::blue };

    return 0;
}
```

This means we now have to prefix our enumeration and enumerator names with the name of the scoped region.

```ad-tip
Classes also provide a scope region, and it’s common to put enumerated types related to a class inside the scope region of the class.
```