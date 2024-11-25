- A **function template** is a function-like definition that is used to generate one or more overloaded functions, each with a different set of actual types. This is what will allow us to create functions that can work with many different types.
- When we create our function template, we use placeholder types (also called **type template parameters**, or informally **template types**) for any parameter types, return types, or types used in the function body that we want to be specified later.
- To create a function template, we’re going to do two things:
	- First, we’re going to replace our specific types with type template parameters.
	- Second, we’re going to tell the compiler that this is a function template, and that `T` is a type template parameter that is a placeholder for any type. This is done using what is called a **template parameter declaration**.
```cpp
template <typename T> // this is the template parameter declaration
T max(T x, T y) // this is the function template definition for max<T>
{
    return (x < y) ? y : x;
}
```

---
- To use our `max<T>` function template, we can make a function call with the following syntax:
```cpp
max<actual_type>(arg1, arg2); // actual_type is some actual type, like int or double
```

````ad-example
```cpp
#include <iostream>

template <typename T>
T max(T x, T y)
{
    return (x < y) ? y : x;
}

int main()
{
    std::cout << max<int>(1, 2) << '\n'; // instantiates and calls function max<int>(int, int)

    return 0;
}
````

- The process of creating functions (with specific types) from function templates (with template types) is called **function template instantiation** (or **instantiation** for short).
```ad-important
The instantiated function will only exist in the file which use the function.
Functions implicitly instantiated from templates are implicitly inline.
[[Using function templates in multiple files]]
```
- When a function is instantiated due to a function call, it’s called **implicit instantiation**.
- A function that is instantiated from a template is technically called a **specialization**, but in common language is often called a **function instance**.
- The template from which a specialization is produced is called a **primary template**.

---
- In cases where the type of the arguments match the actual type we want, we do not need to specify the actual type -- instead, we can use **template argument deduction** to have the compiler deduce the actual type that should be used from the argument types in the function call.
```cpp
std::cout << max<>(1, 2) << '\n';
std::cout << max(1, 2) << '\n';
```

```ad-tip
Best practice:
Favor the normal function call syntax when making calls to a function instantiated from a function template (unless you need the function template version to be preferred over a matching non-template function).
```