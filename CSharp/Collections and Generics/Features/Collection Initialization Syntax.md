- Collection initialization syntax is a language feature which makes it possible to populate many containers (such as ArrayList or List<T\>) with items by using  syntax similar to what you use to populate a basic array.
```ad-note
You can apply collection initialization syntax only to classes that support an Add() method, which is formalized by the ICollection\<T>/ICollection interfaces.
```

```ad-example
````csharp
// Init a standard array.  
int[] myArrayOfInts = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };  
// Init a generic List<> of ints.  
List<int> myGenericList = new List<int> { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };  
// Init an ArrayList with numerical data.  
ArrayList myList = new ArrayList { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
```
- If your container is managing a collection of classes or a structure, you can blend object initialization  syntax with collection initialization syntax to yield some functional code.
```csharp
List<Point> myListOfPoints = new List<Point>  
{  
	new Point { X = 2, Y = 2 },  
	new Point { X = 3, Y = 3 },  
	new Point { X = 4, Y = 4 }  
};
```