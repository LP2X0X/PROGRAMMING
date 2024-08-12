- Using [[Indices and Ranges|indices and ranges]], in the Take() method has been added in .NET 6/C# 10, enabling paging without needing the Take() and Skip() methods used together.

### Example
```csharp
Console.WriteLine("Paging Operations");  
IEnumerable<ProductInfo> list = (from p in products select p).Take(..3);  
OutputResults("The first 3",list);  
list = (from p in products select p).Take(3..);  
OutputResults("Skipping the first 3",list);  
list = (from p in products select p).Take(^2..);  
OutputResults("The last 2",list);  
list = (from p in products select p).Take(3..5);  
OutputResults("Skip 3 take 2",list);  
list = (from p in products select p).Take(..^2);  
OutputResults("Skip the last 2",list);
```