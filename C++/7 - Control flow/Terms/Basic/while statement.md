- A `while statement` is declared using the **while** keyword. When a while-statement is executed, the expression _condition_ is evaluated. If the condition evaluates to `true`, the associated _statement_ executes.
```cpp
while (condition)
    statement;
```
- However, unlike an if-statement, once the statement has finished executing, control returns to the top of the while-statement and the process is repeated. This means a while-statement will keep looping as long as the condition continues to evaluate to `true`.

```ad-tip
Best practice:
Favor `while(true)` for intentional infinite loops.
```