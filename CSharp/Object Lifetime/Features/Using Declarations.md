- New in C# 8.0 is the addition of using declarations. A using declaration is a variable declaration preceded by the using keyword.
```csharp
private static void UsingDeclaration()  
{  
	//This variable will be in scope until the end of the method  
	using var rw = new MyResourceWrapper();  
	//Do something here  
	Console.WriteLine("About to dispose.");  
	//Variable is disposed at this point.  
}
```