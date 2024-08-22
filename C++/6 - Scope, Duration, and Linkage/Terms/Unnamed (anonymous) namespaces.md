- An **unnamed namespace** (also called an **anonymous namespace**) is a namespace that is defined without a name, like so:

```cpp
#include <iostream>

namespace // unnamed namespace
{
    void doSomething() // can only be accessed in this file
    {
        std::cout << "v1\n";
    }
}

int main()
{
    doSomething(); // we can call doSomething() without a namespace prefix

    return 0;
}
```

- All content declared in an unnamed namespace is treated as if it is part of the parent namespace. This might make unnamed namespaces seem useless. But the other effect of unnamed namespaces is that all identifiers inside an unnamed namespace are treated as if they have internal linkage, which means that the content of an unnamed namespace can’t be seen outside of the file in which the unnamed namespace is defined. For functions, this is effectively the same as defining all functions in the unnamed namespace as static functions.
- Unnamed namespaces are typically used when you have a lot of content that you want to ensure stays local to a given translation unit, as it’s easier to cluster such content in a single unnamed namespace than individually mark all declarations as `static`.

```ad-tip
Unnamed namespaces should generally not be used in header files (as every translation unit that \#includes that header will get its own copy of the namespaced content, leading to code bloat).
```