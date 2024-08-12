Two new aggregate functions introduced in .NET 6/C# 10 are MaxBy() and MinBy(). Using the ProductInfo list from earlier, here is a method that gets the max and min, using the NumberInStock property:  

```csharp
static void AggregateOpsBySelector(ProductInfo[] products)  
{  
	Console.WriteLine("Max by In Stock: {0}", products.MaxBy(x=>x.NumberInStock));  
	Console.WriteLine("Min temp: {0}", products.MinBy(x=>x.NumberInStock));  
}
```
