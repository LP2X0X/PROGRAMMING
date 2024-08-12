- As you have seen in this chapter’s initial examples, a query expression can take an orderby operator to sort items in the subset by a specific value. By default, the order will be ascending; thus, ordering by a string would be alphabetical, ordering by numerical data would be lowest to highest, and so forth. If you need to view the results in descending order, simply include the descending operator. Ponder the following method:  

```csharp
static void AlphabetizeProductNames(ProductInfo[] products)  
{  
	// Get names of products, alphabetized.  
	var subset = from p in products orderby p.Name select p;  
	Console.WriteLine("Ordered by Name:");  
	
	foreach (var p in subset)  
	{  
		Console.WriteLine(p.ToString());  
	}  
}  
```

- Although ascending order is the default, you can make your intentions clear by using the ascending operator.  

```csharp
var subset = from p in products orderby p.Name ascending select p;  
```

- If you want to get the items in descending order, you can do so via the descending operator.  

```csharp
var subset = from p in products orderby p.Name descending select p;
```