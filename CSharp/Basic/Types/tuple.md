They are really designed to just be a lightweight data transport mechanism.

## Getting Started with Tuples
To create a tuple, simply enclose the values to be assigned to the tuple in parentheses, as follows:  
```csharp
("a", 5, "c")  
```
Notice that they do not all have to be the same data type.
By default, the compiler assigns each property the name ItemX, where X represents the one-based position in the tuple. For the previous example, the property names are Item1, Item2, and Item3. Accessing them is done as follows:  
```csharp
Console.WriteLine($"First item: {values.Item1}");  
Console.WriteLine($"Second item: {values.Item2}");  
Console.WriteLine($"Third item: {values.Item3}");
```
Specific names can also be added to each property in the tuple on either the right side or the left side of the statement. While it is not a compiler error to assign names on both sides of the statement, if you do, the right side will be ignored, and only the left-side names are used. The following two lines of code show setting the names on the left and the right to achieve the same end:  
```csharp
(string FirstLetter, int TheNumber, string SecondLetter) valuesWithNames = ("a", 5, "c");  
var valuesWithNames2 = (FirstLetter: "a", TheNumber: 5, SecondLetter: "c");  
```
Now the properties on the tuple can be accessed using the field names as well as the ItemX notation, as shown in the following code:  
```csharp
Console.WriteLine($"First item: {valuesWithNames.FirstLetter}");  
Console.WriteLine($"Second item: {valuesWithNames.TheNumber}");  
Console.WriteLine($"Third item: {valuesWithNames.SecondLetter}");  
//Using the item notation still works!  
Console.WriteLine($"First item: {valuesWithNames.Item1}");  
Console.WriteLine($"Second item: {valuesWithNames.Item2}");  
Console.WriteLine($"Third item: {valuesWithNames.Item3}");
```
## Using Inferred Variable Names (Updated 7.1)  
An update to tuples in C# 7.1 is the ability for C# to infer the variable names of tuples, as shown here:  
```csharp
Console.WriteLine("=> Inferred Tuple Names");  
var foo = new {Prop1 = "first", Prop2 = "second"};  
var bar = (foo.Prop1, foo.Prop2);  // Here, the name is not explicitly declare for bar's property
Console.WriteLine($"{bar.Prop1};{bar.Prop2}"); // Still, we can use it
```
## Understanding Tuple Equality/Inequality (New 7.3)
An added feature in C# 7.1 is the tuple equality () and inequality (!=). When testing for inequality the comparison operators will perform implicit conversions on data types within the tuples, including comparing nullable and non-nullable tuples and/or properties. That means the following tests work perfectly, despite the difference between int/long:
```csharp
Console.WriteLine("=> Tuples Equality/Inequality");
// lifted conversions
var left = (a: 5, b: 10);
(int? a, int? b) nullableMembers = (5, 10);
Console.WriteLine(left == nullableMembers); // Also true
// converted type of left is (long, long)
(long a, long b) longTuple = (5, 10);
Console.WriteLine(left == longTuple); // Also true
// comparisons performed on (long, long) tuples
(long a, int b) longFirst = (5, 10);
(int a, long b) longSecond = (5, 10);
Console.WriteLine(longFirst == longSecond); // Also true
```
Tuples that contain tuples can also be compared, but only if they have the same shape. You cannot compare one tuple of three int properties with another tuple of two ints and a tuple.