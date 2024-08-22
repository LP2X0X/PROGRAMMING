- A **using declaration** allows us to use an [[Qualified and unqualified names|unqualified name]] (with no scope) as an alias for a qualified name.

```cpp
#include <iostream>

int main()
{
   using std::cout; // this using declaration tells the compiler that cout should resolve to std::cout
   cout << "Hello world!\n"; // so no std:: prefix is needed here!

   return 0;
} // the using declaration expires at the end of the current scope
```

```ad-info
The using-declaration is active from the point of declaration to the end of the scope in which it is declared.
```