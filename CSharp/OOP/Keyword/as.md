- C# provides the as keyword to quickly determine at runtime whether a given type is compatible with another. When you use the as keyword, you are able to determine compatibility by checking against a null return value.
```ad-example
Here, you loop over each item in the array of objects, checking each one for compatibility with the Hexagon class. If (and only if!) you find a Hexagon-compatible object, you invoke the Draw() method. Otherwise, you simply report the items are not compatible.
````csharp
// Use "as" to test compatibility.  
object[] things = new object[4];  
things[0] = new Hexagon();  
things[1] = false;  
things[2] = new Manager();  
things[3] = "Last thing";  

foreach (object item in things)  
{  
	Hexagon h = item as Hexagon;  
	if (h == null)  
	{  
		Console.WriteLine("Item is not a hexagon");  
	}  
	else  
	{  
		h.Draw();  
	}  
}
```