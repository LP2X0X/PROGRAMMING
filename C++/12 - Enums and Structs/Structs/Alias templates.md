- Creating a type alias for a class template where all template arguments are explicitly specified works just like a normal type alias:

```cpp
#include <iostream>

template <typename T>
struct Pair
{
    T first{};
    T second{};
};

template <typename T>
void print(const Pair<T>& p)
{
    std::cout << p.first << ' ' << p.second << '\n';
}

int main()
{
    using Point = Pair<int>; // create normal type alias
    Point p { 1, 2 };        // compiler replaces this with Pair<int>

    print(p);

    return 0;
}
```

- Such aliases can be defined locally (e.g. inside a function) or globally.

---

### Alias templates

An **alias template** in C++ allows you to create a new name (alias) for a type, simplifying the use of complex templates or providing user-friendly names. Alias templates are defined using the `using` keyword and were introduced in **C++11**.

---

### **Basic Syntax**

The general form of an alias template is:

```cpp
template<typename... Args>
using AliasName = ExistingType<Args...>;
```

Here:

- `Args...` represents template parameters (e.g., types, non-type template parameters).
- `AliasName` is the new name for the specified type or template.

---

### **Examples of Alias Templates**

#### Example 1: Simplifying a Template Type

```cpp
#include <vector>
#include <string>

// Define an alias template for std::vector of any type
template<typename T>
using Vec = std::vector<T>;

int main() {
    Vec<int> intVec;     // Same as std::vector<int>
    Vec<std::string> strVec; // Same as std::vector<std::string>

    intVec.push_back(42);
    strVec.push_back("Hello");

    return 0;
}
```

---

#### Example 2: Parameterizing a Nested Type

```cpp
#include <map>
#include <string>

// Define an alias template for a map from std::string to any type
template<typename T>
using StringMap = std::map<std::string, T>;

int main() {
    StringMap<int> intMap;          // Same as std::map<std::string, int>
    StringMap<std::string> strMap; // Same as std::map<std::string, std::string>

    intMap["one"] = 1;
    strMap["greeting"] = "Hello";

    return 0;
}
```

---

#### Example 3: Using Non-Type Template Parameters

```cpp
#include <array>

// Define an alias template for std::array
template<typename T, std::size_t N>
using Array = std::array<T, N>;

int main() {
    Array<int, 5> arr = {1, 2, 3, 4, 5}; // Same as std::array<int, 5>
    return 0;
}
```

---

### **Advantages of Alias Templates**

1. **Readability:** Simplifies complex template declarations.
2. **Convenience:** Reduces repetitive typing for commonly used types.
3. **Flexibility:** Provides shorter or more meaningful names.

---

### **Comparison with `typedef`**

Before C++11, `typedef` was used to define type aliases. However, `typedef` has limitations when dealing with templates.

#### Example: Using `typedef` vs `using`

```cpp
#include <vector>

// With typedef (cannot handle templates)
typedef std::vector<int> IntVec;

// With using (supports templates)
template<typename T>
using Vec = std::vector<T>;
```

With `typedef`, you cannot define templates, but with `using` and alias templates, you can.

---

### **Combining Alias Templates with SFINAE**

Alias templates can work with **SFINAE** (Substitution Failure Is Not An Error) and concepts in modern C++.

#### Example: Restricting a Type with Alias Templates

```cpp
#include <type_traits>

// Define an alias template for integral types only
template<typename T>
using EnableIfIntegral = std::enable_if_t<std::is_integral_v<T>>;

template<typename T, typename = EnableIfIntegral<T>>
void printIntegral(T value) {
    std::cout << "Value: " << value << '\n';
}

int main() {
    printIntegral(42);       // Works: int is integral
    // printIntegral(3.14);  // Error: double is not integral
    return 0;
}
```

---

### **Common Use Cases**

1. **Simplifying Standard Library Types:**
    
    - Replace long template declarations with aliases, e.g., `std::map<std::string, T>` → `StringMap<T>`.
2. **Customizing Complex Types:**
    
    - Create customized containers or adaptors.
3. **Working with Type Traits:**
    
    - Combine `std::enable_if_t` or other type traits with alias templates.
4. **Reducing Boilerplate in Large Codebases:**
    
    - Provide meaningful type aliases for domain-specific data types.

---

### **Limitations**

1. Alias templates only create new names for existing types or templates—they cannot define entirely new functionality.
2. They rely on existing templates and types to operate.

---

### **Conclusion**

Alias templates are a powerful tool for improving readability, reducing boilerplate code, and simplifying complex template declarations in C++. By leveraging alias templates, you can make your code cleaner, more concise, and easier to maintain.