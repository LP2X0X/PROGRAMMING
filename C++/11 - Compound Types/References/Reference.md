- In C++, a reference is an alias for an existing object. Once a reference has been defined, any operation on the reference is applied to the object being referenced. This means we can use a reference to read or modify the object being referenced.

---
### Lvalue references types
- An **lvalue reference** (commonly just called a “reference” since prior to C++11 there was only one type of reference) acts as an alias for an existing lvalue (such as a variable).
```cpp
// regular types
int        // a normal int type (not an reference)
int&       // an lvalue reference to an int object
double&    // an lvalue reference to a double object
const int& // an lvalue reference to a const int object
```
- There are two kinds of lvalue references:
	- An lvalue reference to a non-const is commonly just called an “lvalue reference”, but may also be referred to as an **lvalue reference to non-const** or a **non-const lvalue reference** (since it isn’t defined using the `const` keyword).
	- An lvalue reference to a const is commonly called either an **lvalue reference to const** or a **const lvalue reference**.
- In most cases, a reference will only bind to an object whose type matches the referenced type.
- Once initialized, a reference in C++ cannot be **reseated**, meaning it cannot be changed to reference another object.

---

### References aren't objects
- Perhaps surprisingly, references are not objects in C++. A reference is not required to exist or occupy storage. If possible, the compiler will optimize references away by replacing all occurrences of a reference with the referent. However, this isn’t always possible, and in such cases, references may require storage.
- This also means that the term “reference variable” is a bit of a misnomer, as variables are objects with a name, and references aren’t objects.
- Because references aren’t objects, they can’t be used anywhere an object is required (e.g. you can’t have a reference to a reference, since an lvalue reference must reference an identifiable object).

---

### Lvalue reference to const
- By using the `const` keyword when declaring an lvalue reference, we tell an lvalue reference to treat the object it is referencing as const. Such a reference is called an **lvalue reference to a const value** (sometimes called a **reference to const** or a **const reference**).
```ad-note
Lvalue references to const can even bind to values of a different type, so long as those values can be implicitly converted to the reference type.
```