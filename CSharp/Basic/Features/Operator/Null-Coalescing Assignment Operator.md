Building on the null-coalescing operator, C# 8 introduced the null-coalescing assignment operator (??=). This operator assigns the left-hand side to the right-hand side only if the left-hand side is null.

```csharp
//Null-coalescing assignment operator  
int? nullableInt = null;  
nullableInt ??= 12;  
nullableInt ??= 14;  
Console.WriteLine(nullableInt);  
```
The nullableInt variable is initialized to null. The next line assigns the value of 12 to the variable since the left-hand side is indeed null. The next line does not assign 14 to the variable since it is not null.