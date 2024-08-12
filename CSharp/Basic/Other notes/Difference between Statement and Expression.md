In programming, the terms "statement" and "expression" are fundamental concepts, but they represent different constructs. Understanding the distinction between the two is crucial for writing and comprehending code effectively.

### Statements

A **statement** is a complete unit of execution. It performs an action and does not return a value. Statements control the flow of execution in a program and can be as simple as assigning a value to a variable or as complex as a loop that iterates over a collection.

Examples of statements in C# include:

- **Declaration Statement**: Declares a variable.
  ```csharp
  int x;
  ```
- **Assignment Statement**: Assigns a value to a variable.
  ```csharp
  x = 10;
  ```
- **Control Flow Statements**: Direct the flow of execution.
  - **If Statement**:
    ```csharp
    if (x > 5)
    {
        Console.WriteLine("x is greater than 5");
    }
    ```
  - **Loop Statement**:
    ```csharp
    for (int i = 0; i < 10; i++)
    {
        Console.WriteLine(i);
    }
    ```
- **Method Invocation Statement**: Calls a method.
  ```csharp
  Console.WriteLine("Hello, world!");
  ```

### Expressions

An **expression** is a construct that evaluates to a value. Expressions are used to compute values and can be composed of operators, variables, literals, and method calls. Unlike statements, expressions always produce a result.

Examples of expressions in C# include:

- **Literal Expression**: A constant value.
  ```csharp
  5
  ```
- **Variable Expression**: A reference to a variable's value.
  ```csharp
  x
  ```
- **Operator Expression**: Combines values using operators.
  ```csharp
  x + 10
  ```
- **Method Call Expression**: Calls a method that returns a value.
  ```csharp
  Math.Sqrt(25)
  ```

### Combining Statements and Expressions

While statements and expressions serve different purposes, they are often combined in programming. For instance, an assignment statement includes an expression on the right-hand side of the assignment operator:

```csharp
x = y + 10;
```

Here, `y + 10` is an expression that evaluates to a value, and `x = y + 10` is an assignment statement that assigns this value to `x`.

### Expression Statements

In some programming languages, including C#, an expression can be used as a statement, known as an **expression statement**. This typically involves expressions that have side effects, such as method calls:

```csharp
Console.WriteLine("Hello, world!");
```

In this case, the method call `Console.WriteLine("Hello, world!")` is an expression that produces no value but has the side effect of printing to the console, and it is used as a statement.

### Summary

- **Statements**: Perform actions, control the flow of execution, and do not return values (e.g., `if`, `for`, variable declarations, and assignments).
- **Expressions**: Evaluate to values and can be used in statements or combined to form more complex expressions (e.g., literals, variables, method calls, and operators).

Understanding these concepts helps you write clear and effective code, as you can properly control program flow and compute values as needed.