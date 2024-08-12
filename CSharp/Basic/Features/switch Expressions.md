Switch expressions are a concise and expressive way of writing switch statements in C# introduced in version 8.0. They offer a more functional and less verbose syntax compared to traditional switch statements.

Here's a basic example to illustrate how switch expressions work:

```csharp
string result = GetDayOfWeek(DayOfWeek.Monday);

string GetDayOfWeek(DayOfWeek day)
{
    return day switch
    {
        DayOfWeek.Monday => "It's Monday!",
        DayOfWeek.Tuesday => "It's Tuesday!",
        DayOfWeek.Wednesday => "It's Wednesday!",
        DayOfWeek.Thursday => "It's Thursday!",
        DayOfWeek.Friday => "It's Friday!",
        DayOfWeek.Saturday => "It's Saturday!",
        DayOfWeek.Sunday => "It's Sunday!",
        _ => throw new ArgumentException("Invalid day of the week.")
    };
}

Console.WriteLine(result); // Output: It's Monday!
```

In this example:
- The `switch` keyword is followed by the expression to evaluate (`day` in this case).
- Each case is followed by the `=>` symbol and the value to return if the expression matches the case.
- The `_` underscore represents the default case, similar to the `default` case in a traditional switch statement.

Switch expressions can also be used to return values directly, making them suitable for assignments:

```csharp
DayOfWeek day = DayOfWeek.Monday;
string message = day switch
{
    DayOfWeek.Monday => "It's the start of the week!",
    DayOfWeek.Friday => "TGIF!",
    _ => "Just another day."
};
Console.WriteLine(message); // Output: It's the start of the week!
```

Switch expressions are particularly useful when you want to assign a value to a variable or return a value from a method based on the value of an expression. They're concise, readable, and offer more flexibility compared to traditional switch statements.