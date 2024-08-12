![[Pasted image 20240617134858.png|center|700]]

### Example
```csharp
public class Point
{
	...
	// Overloaded operator +.
	public static Point operator + (Point p1, Point p2)
		=> new Point(p1.X + p2.X, p1.Y + p2.Y);
	// Overloaded operator -.
	public static Point operator - (Point p1, Point p2)
		=> new Point(p1.X - p2.X, p1.Y - p2.Y);
}
```

In C#, operator overloading allows you to define custom implementations for operators such as `+`, `-`, `*`, `/`, `==`, `!=`, `>`, `<`, and so on. This feature lets you specify how operators should behave with user-defined types (classes and structs) in addition to their standard behavior with built-in types.

Here's how you can overload operators in C#:

### Syntax

Operator overloading is achieved by defining a special type of method inside a class or struct, using the `operator` keyword followed by the operator you want to overload. The method must be marked as `public` and `static`. Here's a general syntax for overloading an operator:

```csharp
public static ReturnType operator Symbol(Type operand1, Type operand2)
{
    // Implementation of operator
}
```

Where:
- `ReturnType` is the type that the operator will return.
- `Symbol` is the operator you want to overload (`+`, `-`, `*`, `/`, `==`, `!=`, etc.).
- `Type` is the type of the operands.

### Example

Let's say you have a `Vector` class and you want to overload the `+` operator to add two vectors:

```csharp
public class Vector
{
    public int X { get; set; }
    public int Y { get; set; }

    public Vector(int x, int y)
    {
        X = x;
        Y = y;
    }

    public static Vector operator +(Vector v1, Vector v2)
    {
        return new Vector(v1.X + v2.X, v1.Y + v2.Y);
    }
}
```

In this example:
- We've defined a `Vector` class with properties `X` and `Y`.
- We overloaded the `+` operator using the `operator` keyword, specifying that it should add two `Vector` objects together and return a new `Vector` object with the sum of their `X` and `Y` components.

### Usage

After overloading an operator, you can use it just like any built-in operator:

```csharp
Vector v1 = new Vector(1, 2);
Vector v2 = new Vector(3, 4);
Vector v3 = v1 + v2; // Now v3 will be a Vector with X=4, Y=6
```

### Guidelines

When overloading operators, consider these guidelines:
- Operators must be `public` and `static`.
- They cannot be overridden but can be overloaded.
- Overloaded operators must be implemented as pairs (e.g., if you overload `+`, you should also consider overloading `-` for consistency).
- Avoid overloading operators in a way that makes the code less readable or violates expected behavior.

Operator overloading in C# can be powerful but should be used judiciously to maintain code clarity and consistency.