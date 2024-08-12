- The final unsafe-centric C# keyword to consider is sizeof. As in C++, the C# sizeof keyword is used to obtain the size in bytes of an intrinsic data type, but not a custom type, unless within an unsafe context.

```csharp
unsafe static void UseSizeOfOperator()  
{  
	unsafe {  
		Console.WriteLine("The size of Point is {0}.", sizeof(Point));  
	}  
}
```