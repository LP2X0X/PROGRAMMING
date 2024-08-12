```csharp
namespace CustomInterfaces;  
interface IRegularPointy : IPointy  
{  
	int SideLength { get; set; }  
	int NumberOfSides { get; set; }  
	int Perimeter => SideLength * NumberOfSides;  
}

namespace CustomInterfaces;  
class Square: Shape,IRegularPointy  
{  
	public Square() { }  
	public Square(string name) : base(name) { }  
	//Draw comes from the Shape base class  
	public override void Draw()  
	{  
		Console.WriteLine("Drawing a square");  
	}  
	//This comes from the IPointy interface  
	public byte Points => 4;
	
	//These come from the IRegularPointy interface  
	public int SideLength { get; set; }  
	public int NumberOfSides { get; set; }  
	//Note that the Perimeter property is not implemented  
}
```

- In the above example, we implement the IRegularPointy interface which inherits from IPointy interface. Here the Perimeter property has a default implementations. The Perimeter property, defined on the IRegularPointy interface, is not defined in the Square class, making it **inaccessible from an instance of Square**.

```csharp
Console.WriteLine("\n***** Fun with Interfaces *****\n");  
...  
var sq = new Square("Boxy")  
	{NumberOfSides = 4, SideLength = 4};  
sq.Draw();  
//This won't compile  
//Console.WriteLine($"{sq.PetName} has {sq.NumberOfSides} of length {sq.SideLength} and a  
perimeter of {sq.Perimeter}");
```

- The Square instance must be explicitly cast to the IRegularPointy interface then the Perimeter property can be accessed.

```csharp
Console.WriteLine($"{sq.PetName} has {sq.NumberOfSides} of length {sq.SideLength} and a perimeter of {((IRegularPointy)sq).Perimeter}");
```

- One option to get around this problem is to always code to the interface of a type. Change the definition of the Square instance to IRegularPointy instead of Square, like this:  

```csharp
IRegularPointy sq = new Square("Boxy") {NumberOfSides = 4, SideLength = 4};  
```

- The problem with this approach (in our case) is that the Draw() method and PetName property are not defined on the interface, causing compilation errors.