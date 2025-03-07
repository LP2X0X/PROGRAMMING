#### **What Are Static Member Variables?**

- **Shared across all objects** of a class (not unique to each object).
- **Stored in a single memory location**, not within individual objects.
- **Declared using `static`** inside the class but **must be defined outside**.
	- This is because when you declare a static member variable inside the class, you're only **declaring its existence**. This tells the compiler that the variable **belongs to the class** but does **not allocate memory** for it.

```ad-note
Note that this static member definition is not subject to access controls: you can define and initialize the value even if it’s declared as private (or protected) in the class (as definitions are not considered to be a form of access).
```

---

#### **Example Usage**

```cpp
#include <iostream>

class Example {
public:
    static int count; // Declaration (shared by all instances)

    Example() { ++count; } // Increases count when an object is created
};

// Definition outside the class (required)
int Example::count = 0;

int main() {
    Example obj1, obj2, obj3;
    std::cout << Example::count; // Output: 3
}
```

**Explanation:**

- `count` is **shared** by all instances.
- When an object is created, `count` increases.

---

#### **Key Properties**

|Feature|Description|
|---|---|
|**Shared**|One copy for all objects.|
|**Scope**|Exists even if no object is created.|
|**Initialization**|Must be defined **outside** the class.|
|**Access**|Use `ClassName::variable` (e.g., `Example::count`).|

---

#### **When to Use Static Variables?**

- **Tracking object count** (e.g., `InstanceCounter`).
- **Managing shared resources** (e.g., database connections).
- **Caching values** to optimize performance.

---

#### **Static Inside Functions**

- Retains value **between function calls**.
- Initialized only **once**.

```cpp
void demo() {
    static int count = 0; // Value persists across calls
    ++count;
    std::cout << count << std::endl;
}

int main() {
    demo(); // Output: 1
    demo(); // Output: 2
}
```

---

#### **Initialization of static member variables inside the class definition**

There are a few shortcuts to the above. First, when the static member is a constant integral type (which includes `char` and `bool`) or a const enum, the static member can be initialized inside the class definition:

```cpp
class Whatever
{
public:
    static const int s_value{ 4 }; // a static const int can be defined and initialized directly
};
```

In the above example, because the static member variable is a const int, no explicit definition line is needed. This shortcut is allowed because these specific const types are compile-time constants.


```cpp
class Whatever
{
public:
    static inline int s_value{ 4 }; // a static inline variable can be defined and initialized directly
};
```

Such variables can be initialized inside the class definition regardless of whether they are constant or not. This is the preferred method of defining and initializing static members.

Because `constexpr` members are implicitly inline (as of C++17), static `constexpr` members can also be initialized inside the class definition without explicit use of the `inline` keyword:

```cpp
#include <string_view>

class Whatever
{
public:
    static constexpr double s_value{ 2.2 }; // ok
    static constexpr std::string_view s_view{ "Hello" }; // this even works for classes that support constexpr initialization
};
```

```ad-note
Best practice
Make your static members `inline` or `constexpr` so they can be initialized inside the class definition.
```

---
---

### **Initializing Static Member Variables Inside a Class**

Usually, **static member variables** are defined **outside the class**, but there are some shortcuts that allow them to be initialized **inside** the class definition.

---

## **1. Static `const int` or `const enum` Can Be Initialized Inside the Class**

If the static member is a **constant integral type** (like `int`, `char`, `bool`, or an `enum`), you can **initialize it directly inside the class**.

✅ **Example:**

```cpp
class Example {
public:
    static const int value{ 4 }; // Allowed because it's a constant int
};
```

🔹 **Why does this work?**  
Because `const int` is a **compile-time constant**, the compiler allows this shortcut. **No need** to define it separately outside the class.

---

## **2. Using `inline` (C++17 and Later)**

If you're using **C++17 or later**, you can use `inline` for **any** static variable, whether it's constant or not.

✅ **Example:**

```cpp
class Example {
public:
    static inline int value{ 4 }; // `inline` allows direct initialization
};
```

🔹 **Why use `inline`?**

- It **removes the need** to define the variable separately outside the class.
- Works for **both constants and non-constants**.
- Avoids multiple definition errors when using the class across multiple files.

---

## **3. `constexpr` Static Members Are Implicitly `inline` (C++17 and Later)**

If a static variable is **`constexpr`**, it is **automatically inline**, meaning you can initialize it inside the class.

✅ **Example:**

```cpp
#include <string_view>

class Example {
public:
    static constexpr double pi{ 3.14159 }; // Works fine
    static constexpr std::string_view message{ "Hello" }; // Works for `constexpr`-compatible classes
};
```

🔹 **Why does this work?**

- `constexpr` means the value **must be known at compile time**.
- `constexpr` variables are automatically `inline`, so they don't need a separate definition.
- Works even for **some class types** like `std::string_view`.

---

## **🛠️ Best Practice Recommendation**

✅ **Use `inline` or `constexpr`** to define and initialize static members **inside the class** whenever possible.  
🔹 This makes the code **cleaner**, **simpler**, and **easier to manage**.

Would you like a comparison of the old vs. new way of doing this? 🚀
### **Final Thoughts**

✅ **Use static variables** for **shared data** that belongs to the class, not individual objects.  
✅ Remember to **define them outside the class**.  
✅ Great for **counters, caching, and singleton patterns**.
