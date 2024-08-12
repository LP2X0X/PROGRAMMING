- When a class (or structure) chooses to extend its functionality by supporting interfaces, it does so using a comma-delimited list in the type definition. Be aware that the direct base class must be the first item listed after the colon operator. When your class type derives directly from System.Object, you are free to simply list the interface (or interfaces) supported by the class, as the C# compiler will extend your types from System.Object if you do not say otherwise.
- Understand that implementing an interface is an all-or-nothing proposition for interface items that do not include a default implementation. The supporting type is not able to selectively choose which members it will implement.

---

- One way to determine at runtime whether a type supports a specific interface is to use an explicit cast. If the type does not support the requested interface, you receive an InvalidCastException.
```csharp
// Catch a possible InvalidCastException.
Circle c = new Circle("Lisa");
IPointy itfPt = null;
try
{
	itfPt = (IPointy)c;
	Console.WriteLine(itfPt.Points);
}
catch (InvalidCastException e)
{
	Console.WriteLine(e.Message);
}
Console.ReadLine();
```

</br>

- You can determine whether a given type supports an interface by using the as keyword, introduced in Chapter 6. If the object can be treated as the specified interface, you are returned a reference to the interface in question. If not, you receive a null reference. Therefore, be sure to check against a null value before proceeding.  

```csharp
Hexagon hex2 = new Hexagon("Peter");  
IPointy itfPt2 = hex2 as IPointy;  
if (itfPt2 != null)  
{  
	Console.WriteLine("Points: {0}", itfPt2.Points);  
}  
else  
{  
	Console.WriteLine("OOPS! Not pointy...");  
}  
Console.ReadLine();
```

</br>

- You may also check for an implemented interface using the is keyword (also first discussed in Chapter 6). If the object in question is not compatible with the specified interface, you are returned the value false. If you supply a variable name in the statement, the type is assigned into the variable, eliminating the need to do the type check and perform a cast.

```csharp
Console.WriteLine("***** Fun with Interfaces *****\n");  
...  
if (hex2 is IPointy itfPt3)  
{  
	Console.WriteLine("Points: {0}", itfPt3.Points);  
}  
else  
{  
	Console.WriteLine("OOPS! Not pointy...");  
}  
Console.ReadLine();
```