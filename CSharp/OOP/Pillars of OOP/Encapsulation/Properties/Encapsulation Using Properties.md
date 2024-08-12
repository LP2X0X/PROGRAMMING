- Although you can encapsulate a piece of field data using traditional get and set methods, .NET Core languages prefer to enforce data encapsulation state data using properties. First, understand that properties are just a container for “real” accessor and mutator methods, named get and set, respectively. Therefore, as a class designer, you are still able to perform any internal logic necessary before making the value assignment (e.g., uppercase the value, scrub the value for illegal characters, check the bounds of a numerical value, etc.).
## Using Properties Within a Class Definition
- Properties, specifically the set portion of a property, are common places to package up the business rules of your class. Currently, the Employee class has a Name property that ensures the name is no more than 15 characters. The remaining properties (ID, Pay, and Age) could also be updated with any relevant logic.  
- While this is well and good, also consider what a class constructor typically does internally. It will take the incoming parameters, check for valid data, and then make assignments to the internal private fields. Currently, your master constructor does not test the incoming string data for a valid range, so you could update this member as so:
```csharp
public Employee(string name, int age, int id, float pay)  
{  
	// Humm, this seems like a problem...  
	if (name.Length > 15)  
	{  
		Console.WriteLine("Error! Name length exceeds 15 characters!");  
	}  
	else  
	{  
		_empName = name;  
	}  
	_empId = id;  
	_empAge = age;  
	_currPay = pay;  
}
```
- I am sure you can see the problem with this approach. The Name property and your master constructor are performing the same error checking. If you were also making checks on the other data points, you would have a good deal of duplicate code. To streamline your code and isolate all of your error checking to a central location, you will do well if you always use properties within your class whenever you need to get or set the values. Consider the following updated constructor:
```csharp
public Employee(string name, int age, int id, float pay)
{
	// Better! Use properties when setting class data.
	// This reduces the amount of duplicate error checks.
	Name = name;
	Age = age;
	ID = id;
	Pay = pay;
}
```
- Beyond updating constructors to use properties when assigning values, it is good practice to use properties throughout a class implementation to ensure your business rules are always enforced. In many cases, the only time when you directly refer to the underlying private piece of data is within the property itself.