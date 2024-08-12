- When you call the Concat() extension method, you could very well end up with redundant entries in the fetched result, which could be exactly what you want in some cases. However, in other cases, you might want to remove duplicate entries in your data. To do so, simply call the Distinct() extension method, as shown here:

```csharp
static void DisplayConcatNoDups()  
{  
	List<string> myCars =  
	new List<String> { "Yugo", "Aztec", "BMW" };  

	List<string> yourCars =  
	new List<String> { "BMW", "Saab", "Aztec" };  

	var carConcat =  
	(from c in myCars select c)  
	.Concat(from c2 in yourCars select c2);  

	// Prints:  
	// Yugo Aztec BMW Saab.  
	foreach (string s in carConcat.Distinct())  
	{  
		Console.WriteLine(s);  
	}  
}
```