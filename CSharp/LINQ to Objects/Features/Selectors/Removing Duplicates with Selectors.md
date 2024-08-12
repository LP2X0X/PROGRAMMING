- Another new method in .NET 6/C# 10 for working with lists is the DistinctBy() method. The following example gets the distinct records by Age. Notice that the final list only includes Lindsey of the three tuples that have 30 for their Age value. The accepted tuple is the one that appears first in the list.

### Example
```csharp
static void DisplayConcatNoDupsBySelector()  
{  
	var first = new (string Name, int Age)[] { ("Francis", 20), ("Lindsey", 30),  
	("Ashley", 40) };  
	var second = new (string Name, int Age)[] { ("Claire", 30), ("Pat", 30), ("Drew", 33) };  

	var result = first.Concat(second).DistinctBy(x=>x.Age);  

	Console.WriteLine("Distinct by selector:");  
	foreach (var item in result)  
	{  
		Console.WriteLine(item); // { ("Francis", 20), ("Lindsey", 30), ("Ashley", 40),  
		("Drew", 33) };  
	}  
}
```