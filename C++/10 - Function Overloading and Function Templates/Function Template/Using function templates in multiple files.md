- Consider the following program, which doesn’t work correctly:
main.cpp:

```cpp
#include <iostream>

template <typename T>
T addOne(T x); // function template forward declaration

int main()
{
    std::cout << addOne(1) << '\n';
    std::cout << addOne(2.3) << '\n';

    return 0;
}
```

add.cpp:

```cpp
template <typename T>
T addOne(T x) // function template definition
{
    return x + 1;
}
```

- If `addOne` were a non-template function, this program would work fine: In _main.cpp_, the compiler would be satisfied with the forward declaration of `addOne`, and the linker would connect the call to `addOne()` in _main.cpp_ to the function definition in _add.cpp_.
- But because `addOne` is a template, this program doesn’t work, and we get a linker error:
1>Project6.obj : error LNK2019: unresolved external symbol "int __cdecl addOne\<int>(int)" (??$addOne@H@@YAHH@Z) referenced in function _main
1>Project6.obj : error LNK2019: unresolved external symbol "double __cdecl addOne\<double>(double)" (??$addOne@N@@YANN@Z) referenced in function _main
- In _main.cpp_, we call `addOne<int>` and `addOne<double>`. However, since the compiler can’t see the definition for function template `addOne`, it can’t instantiate those functions inside _main.cpp_. It does see the forward declaration for `addOne` though, and will assume those functions exist elsewhere and will be linked in later.
- When the compiler goes to compile _add.cpp_, it will see the definition for function template `addOne`. However, there are no uses of this template in _add.cpp_, so the compiler will not instantiate anything. The end result is that the linker is unable to connect the calls to `addOne<int>` and `addOne<double>` in _main.cpp_ to the actual functions, because those functions were never instantiated.

---

### Solution
- The most conventional way to address this issue is to put all your template code in a header (.h) file instead of a source (.cpp) file:
add.h:

```cpp
#ifndef ADD_H
#define ADD_H

template <typename T>
T addOne(T x) // function template definition
{
    return x + 1;
}

#endif
```

main.cpp:

```cpp
#include "add.h" // import the function template definition
#include <iostream>

int main()
{
    std::cout << addOne(1) << '\n';
    std::cout << addOne(2.3) << '\n';

    return 0;
}
```

- That way, any files that need access to the template can #include the relevant header, and the template definition will be copied by the preprocessor into the source file. The compiler will then be able to instantiate any functions that are needed.