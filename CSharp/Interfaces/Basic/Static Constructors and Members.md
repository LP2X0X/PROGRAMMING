- Another addition to interfaces in C# 8.0 is the ability to have static constructors and members, which function the same as static members on class definitions but are defined on interfaces.

```csharp
interface IRegularPointy : IPointy  
{  
	int SideLength { get; set; }  
	int NumberOfSides { get; set; }  
	int Perimeter => SideLength * NumberOfSides;
	
	//Static members are also allowed in C# 8  
	static string ExampleProperty { get; set; }  
	static IRegularPointy() => ExampleProperty = "Foo";  
}
```

- Static constructors must be parameterless and can only access static properties and methods.

```ad-note
The static property must be invoked from the interface and not an instance variable.
````csharp
IRegularPointy.ExampleProperty = "Updated";
```