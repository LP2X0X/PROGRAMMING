- Polymorphism provides a way for a subclass to define its own version of a method defined by its base class, using the process termed method overriding.
---
- To retrofit your current design, you need to understand the meaning of the virtual and override keywords. If a base class wants to define a method that may be (but does not have to be) overridden by a subclass, it must mark the method with the virtual keyword.
```csharp
partial class Employee  
{  
	// This method can now be "overridden" by a derived class.  
	public virtual void GiveBonus(float amount)  
	{  
		Pay += amount;  
	}  
	...  
}
```

---
- When a subclass wants to change the implementation details of a virtual method, it does so using the override keyword.
```csharp
//SalesPerson.cs  
namespace Employees;
class SalesPerson : Employee  
{  
...  
// A salesperson's bonus is influenced by the number of sales.  
public override void GiveBonus(float amount)  
{  
	int salesBonus = 0;  
	if (SalesNumber >= 0 && SalesNumber <= 100)  
	{  
	salesBonus = 10;  
	}  
	else  
	{  
		if (SalesNumber >= 101 && SalesNumber <= 200)  
		{  
			salesBonus = 15;  
		}  
		else  
		{  
			salesBonus = 20;  
		}  
	}  
	base.GiveBonus(amount * salesBonus);  
	}  
}  
//Manager.cs  
namespace Employees;  
class Manager : Employee  
{  
...  
	public override void GiveBonus(float amount)  
	{  
		base.GiveBonus(amount);  
		Random r = new Random();  
		StockOptions += r.Next(500);  
	}  
}
```