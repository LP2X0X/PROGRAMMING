- The **for statement** (also called a **for loop**) is preferred when we have an obvious loop variable because it lets us easily and concisely define, initialize, test, and change the value of loop variables:
```cpp
for (init-statement; condition; end-expression)
   statement;
```
- A for-statement is evaluated in 3 parts:
- First, the **init-statemen**t is executed. This only happens once when the loop is initiated.
- Second, with each **loop iteration**, the condition is evaluated. If this evaluates to `true`, the statement is executed. If this evaluates to `false`, the loop terminates and execution continues with the next statement beyond the loop.
- Finally, after the statement is executed, the **end-expression** is evaluated. Typically, this expression is used to increment or decrement the loop variables defined in the init-statement. After the end-expression has been evaluated, execution returns to the second step (and the condition is evaluated again).

```ad-note
The end-expression of a for-loop executes _after_ the loop statement.
```

---

### For-loops with multiple counters

- Although for-loops typically iterate over only one variable, sometimes for-loops need to work with multiple variables. To assist with this, the programmer can define multiple variables in the init-statement, and can make use of the comma operator to change the value of multiple variables in the end-expression:
```cpp
#include <iostream>

int main()
{
    for (int x{ 0 }, y{ 9 }; x < 10; ++x, --y)
        std::cout << x << ' ' << y << '\n';

    return 0;
}
```