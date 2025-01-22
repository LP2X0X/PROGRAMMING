- For non-const member functions, `this` is a const pointer to a non-const value (meaning `this` cannot be pointed at something else, but the object pointing to may be modified). With const member functions, `this` is a const pointer to a const value (meaning the pointer cannot be pointed at something else, nor may the object being pointed to be modified).

```cpp
class Example {
private:
    int value;

public:
    Example(int v) : value(v) {}

    void modify() {
        value = 20; // Allowed, as `this` points to a non-const object
    }

    int getValue() const {
        return value; // Allowed, as `this` points to a const object
    }

    void print() const {
        // value = 30; // Error: Cannot modify the object in a const function
        std::cout << "Value: " << value << std::endl;
    }
};

int main() {
    Example obj(10);

    obj.modify();      // Allowed, non-const function
    obj.print();       // Allowed, const function
    int value = obj.getValue(); // Allowed, const function

    const Example constObj(15); // Const object
    // constObj.modify();       // Error: Cannot call non-const function on const object
    constObj.print();           // Allowed, as `print` is const

    return 0;
}

```