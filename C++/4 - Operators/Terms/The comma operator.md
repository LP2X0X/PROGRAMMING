- The **comma operator (,)** allows you to evaluate multiple expressions wherever a single expression is allowed. The comma operator evaluates the left operand, then the right operand, and then returns the result of the right operand.

| Operator | Symbol | Form | Operation                             |
| -------- | ------ | ---- | ------------------------------------- |
| Comma    | ,      | x, y | Evaluate x then y, returns value of y |

- For example:
```cpp
#include <iostream>

int main()
{
    int x{ 1 };
    int y{ 2 };

    std::cout << (++x, ++y) << '\n'; // increment x and y, evaluates to the right operand

    return 0;
}
```
- First the left operand of the comma operator is evaluated, which increments _x_ from _1_ to _2_. Next, the right operand is evaluated, which increments _y_ from _2_ to _3_. The comma operator returns the result of the right operand (_3_), which is subsequently printed to the console.
</br>
- Note that comma has the lowest precedence of all the operators, even lower than assignment. Because of this, the following two lines of code do different things:
```cpp
z = (a, b); // evaluate (a, b) first to get result of b, then assign that value to variable z.
z = a, b; // evaluates as "(z = a), b", so z gets assigned the value of a, and b is evaluated and discarded.
```