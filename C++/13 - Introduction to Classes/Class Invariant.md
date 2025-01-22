### **Class Invariant in C++**

A **class invariant** is a logical condition that must always hold true for all instances of a class after the completion of any constructor or public method. These invariants ensure that the object remains in a consistent and valid state throughout its lifetime.

---

### **Purpose of Class Invariants**

- Maintain the internal consistency of an object.
- Define valid states for a class.
- Help in debugging and catching logical errors early.
- Support code maintainability by clearly stating the rules for a class.

---

### **How to Define and Check Class Invariants**

1. **Define the Invariant:**  
    Specify conditions that must always be true for the class (e.g., non-negative size, valid ranges, etc.).
    
2. **Enforce the Invariant:**
    
    - Check the invariant at the end of constructors and any public methods that modify the object's state.
    - Use helper methods to encapsulate the invariant-checking logic.
3. **Debugging:**  
    In debug mode, you can assert the invariant using the `assert` function.
    

---

### **Example of Class Invariant**

```cpp
#include <iostream>
#include <cassert>

class Rectangle {
private:
    int width;
    int height;

    // Helper to check class invariant
    void checkInvariant() const {
        assert(width > 0 && "Width must be positive");
        assert(height > 0 && "Height must be positive");
    }

public:
    // Constructor
    Rectangle(int w, int h) : width(w), height(h) {
        checkInvariant(); // Ensure invariant is met
    }

    // Setters
    void setWidth(int w) {
        width = w;
        checkInvariant(); // Check invariant after modification
    }

    void setHeight(int h) {
        height = h;
        checkInvariant();
    }

    // Area calculation
    int area() const {
        return width * height;
    }
};

int main() {
    Rectangle rect(5, 10); // Valid rectangle
    std::cout << "Area: " << rect.area() << std::endl;

    rect.setWidth(20); // Valid
    rect.setHeight(0); // This will trigger an assertion failure in debug mode

    return 0;
}
```

---

### **Best Practices for Class Invariants**

1. **Keep it Simple:** Define only the minimal set of invariants necessary to ensure the object's correctness.
2. **Encapsulation:** Use private or protected methods to encapsulate invariant-checking logic.
3. **Debug Mode:** Use `assert` to check invariants, as it is disabled in release builds, avoiding performance overhead.
4. **Documentation:** Clearly document the invariants in comments or documentation to ensure maintainability.

By adhering to class invariants, you ensure the reliability and stability of your code, particularly for complex classes with intricate state management.