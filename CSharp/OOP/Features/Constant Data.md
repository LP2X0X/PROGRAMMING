- Regardless of where you define a constant piece of data, the one point to always remember is that the initial value assigned to the constant must be specified at the time you define the constant.
- Constant fields of a class are implicitly static. However, it is permissible to define and access a local constant variable within the scope of a method or property.

## Constant Interpolated Strings
- Introduced in C# 10, const string values can use string interpolation in their assignment statements as long as all of the components that are used are also const strings. As a trivial example, add the following code to the top-level statements:  
```csharp
Console.WriteLine("=> Constant String Interpolation:");  
const string foo = "Foo";  
const string bar = "Bar";  
const string foobar = $"{foo}{bar}";  
Console.WriteLine(foobar);
```
