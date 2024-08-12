```csharp
static void PatternMatching<T>(Point<T> p)  
{  
	switch (p)  
	{  
		case Point<string> pString:  
			Console.WriteLine("Point is based on strings");  
			return;  
		case Point<int> pInt:  
			Console.WriteLine("Point is based on ints");  
			return;  
	}  
}
```