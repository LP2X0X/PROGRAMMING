- It is also possible to project new forms of data from an existing data source. Let’s assume you want to take the incoming ProductInfo[] parameter and obtain a result set that accounts only for the name and description of each item. To do so, you can define a select statement that dynamically yields a new anonymous type.

```csharp
var nameDesc =  
	from p  
	in products  
	select new { p.Name, p.Description };
```

---

- In addition to projecting into anonymous types, you can project the results of your LINQ query into another concrete type.

```csharp
IEnumerable<ProductInfoSmall> nameDesc =  
	from p  
	in products  
	select new ProductInfoSmall  
	{ Name=p.Name, Description=p.Description };
```