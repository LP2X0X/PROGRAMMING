- In addition to the as keyword, the C# language provides the is keyword to determine whether two items are compatible. Unlike the as keyword, however, the is keyword returns false, rather than a null reference, if the types are incompatible.
- The is keyword can also assign the converted type to a variable if the cast works. This cleans up the preceding method by preventing the “double-cast” problem.
```csharp
static void GivePromotion(Employee emp)  
{  
	Console.WriteLine("{0} was promoted!", emp.Name);  
	//Check if is SalesPerson, assign to variable s  
	if (emp is SalesPerson s)  
	{  
		Console.WriteLine("{0} made {1} sale(s)!", s.Name, s.SalesNumber);  
		Console.WriteLine();  
	}  
	//Check if is Manager, if it is, assign to variable m  
	else if (emp is Manager m)  
	{  
		Console.WriteLine("{0} had {1} stock options...", m.Name, m.StockOptions);  
		Console.WriteLine();  
	}  
}
```

```csharp
if (emp is not Manager and not SalesPerson)  
{  
	Console.WriteLine("Unable to promote {0}. Wrong employee type", emp.Name);  
	Console.WriteLine();  
}
```

- If you want to create a catchall in your if or switch statement, you can do so as follows:
```csharp
if (obj is var _)  
{  
//do something  
}
```