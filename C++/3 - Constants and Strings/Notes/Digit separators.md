---
tags:
  - cpp
  - note
  - term
  - fundamental
---

- Because long literals can be hard to read, C++14 also adds the ability to use a quotation mark (‘) as a digit separator.

```cpp
#include <iostream>

int main()
{
    int bin { 0b1011'0010 };  // assign binary 1011 0010 to the variable
    long value { 2'132'673'462 }; // much easier to read than 2132673462

    return 0;
}
```

- Also note that the separator can not occur before the first digit of the value:

```cpp
int bin { 0b'1011'0010 };  // error: ' used before first digit of value
```

- Digit separators are purely visual and do not impact the literal value in any way.