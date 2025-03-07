- Within the scope of a class, the unqualified name of the class is called an **injected class name**. In a class template, the injected class name serves as shorthand for the fully templated name.
```cpp
template<typename T>
struct Wrapper
{
    Wrapper() { /* constructor */ }
};
```

- That `Wrapper()` is using the injected class name. The full name of the constructor is

```cpp
Wrapper<T>() { /* constructor */ }
```

- But C++ secretly injects the simple name of the class into the class itself, as if you had written

```cpp
using Wrapper = Wrapper<T>;
```

- Injected class names are public and therefore can be inherited:
```cpp
namespace details {
    struct Base
    {
        Base(int) { /* constructor */ }
    };
}

struct Derived : details::Base
{
    Derived() : Base(42) { /* constructor */ }
}
```
