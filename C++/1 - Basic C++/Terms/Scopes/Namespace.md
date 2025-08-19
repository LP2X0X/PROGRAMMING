---
tags:
  - cpp
  - term
  - fundamental
---

A **namespace** provides another type of [[Scope region]] (called **namespace scope**) that allows you to declare names inside of it for the purpose of disambiguation. Any names declared inside the namespace won’t be mistaken for identical names in other scopes.

```ad-note
A namespace may only contain declarations and definitions. Executable statements are only allowed as part of a definition (e.g. of a function).
```

Here's a basic example of how namespaces work in C++:

### Declaring a Namespace
```cpp
namespace MyNamespace {
    int myVariable = 42;

    void myFunction() {
        // Function code here
    }
}
```

### Accessing Members of a Namespace
To access the members of a namespace, you typically use the [[C++/1 - Basic C++/Terms/Operators/Scope resolution operator|scope resolution operator]].

```cpp
int main() {
    // Accessing a variable from the namespace
    std::cout << MyNamespace::myVariable << std::endl;

    // Calling a function from the namespace
    MyNamespace::myFunction();

    return 0;
}
```

### Using `using` Directive
You can also bring all or some names from a namespace into the current scope with the `using` directive:

```cpp
using namespace MyNamespace;

int main() {
    // Now you can access members without qualifying with the namespace name
    std::cout << myVariable << std::endl;
    myFunction();

    return 0;
}
```

However, using `using namespace` in global scope (especially with large namespaces like `std`) can lead to name conflicts and ambiguity, so it is generally recommended to avoid this practice in header files and large projects.

### Nested Namespaces
Namespaces can be nested:

```cpp
namespace OuterNamespace {
    namespace InnerNamespace {
        void innerFunction() {
            // Function code here
        }
    }
}

int main() {
    OuterNamespace::InnerNamespace::innerFunction();
    return 0;
}
```

### Anonymous Namespaces
An anonymous namespace has no name and is useful when you want to restrict the visibility of functions or variables to the file in which they are declared:

```cpp
namespace {
    int internalVariable = 0;

    void internalFunction() {
        // Function code here
    }
}

int main() {
    internalFunction();
    return 0;
}
```

In this case, `internalVariable` and `internalFunction` are only accessible within the file where the anonymous namespace is declared, providing internal linkage (similar to the `static` keyword).

Namespaces are a powerful feature in C++ that help to keep your code modular and avoid name collisions, especially in large projects or when integrating multiple libraries.

---

### The global namespace
- In C++, any name that is not defined inside a class, function, or a namespace is considered to be part of an implicitly-defined namespace called the **global namespace** (sometimes also called **the global scope**).

### The std namespace
- When C++ was originally designed, all of the identifiers in the C++ standard library (including std::cin and std::cout) were available to be used without the `std::` prefix (they were part of the global namespace). However, this meant that any identifier in the standard library could potentially conflict with any name you picked for your own identifiers (also defined in the global namespace). Code that was once working might suddenly have a naming conflict when you include a different part of the standard library. Or worse, code that compiles under one version of C++ might not compile under the next version of C++, as new identifiers introduced into the standard library could have a naming conflict with already written code. So C++ moved all of the functionality in the standard library into a namespace named `std` (short for “standard”).
- The :: symbol is an operator called the **scope resolution operator**. The identifier to the left of the `::` symbol identifies the namespace that the name to the right of the `::` symbol is contained within. If no identifier to the left of the `::` symbol is provided, the global namespace is assumed.
- When an identifier includes a namespace prefix, the identifier is called a **qualified name**.

```ad-warning
Avoid using-directives (such as `using namespace std;`) at the top of your program or in header files. They violate the reason why namespaces were added in the first place.
```