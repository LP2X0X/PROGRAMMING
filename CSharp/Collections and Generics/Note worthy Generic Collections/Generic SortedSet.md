- The SortedSet\<T> class is useful because it automatically ensures that the items in the set are sorted when you insert or remove items. However, you do need to inform the SortedSet\<T> class exactly how you want it to sort the objects, by passing in as a constructor argument an object that implements the generic IComparer\<T> interface.

```csharp
namespace FunWithGenericCollections;  

class SortPeopleByAge : IComparer<Person>  
{  
	public int Compare(Person firstPerson, Person secondPerson)  
	{  
		if (firstPerson?.Age > secondPerson?.Age)  
	{  
		return 1;  
	}  
	
	if (firstPerson?.Age < secondPerson?.Age)  
	{  
		return -1;  
	}  
		return 0;  
	}  
}
```

```csharp
static void UseSortedSet()  
{  
	// Make some people with different ages.  
	SortedSet<Person> setOfPeople = new SortedSet<Person>(new SortPeopleByAge())  
	{  
		new Person {FirstName= "Homer", LastName="Simpson", Age=47},  
		new Person {FirstName= "Marge", LastName="Simpson", Age=45},  
		new Person {FirstName= "Lisa", LastName="Simpson", Age=9},  
		new Person {FirstName= "Bart", LastName="Simpson", Age=8}  
	};  
	// Note the items are sorted by age!  
	foreach (Person p in setOfPeople)  
	{  
		Console.WriteLine(p);  
	} 
	 
	Console.WriteLine();  
	// Add a few new people, with various ages.  
	setOfPeople.Add(new Person { FirstName = "Saku", LastName = "Jones", Age = 1 });  
	setOfPeople.Add(new Person { FirstName = "Mikko", LastName = "Jones", Age = 32 });  
	// Still sorted by age!  
	foreach (Person p in setOfPeople)  
	{  
		Console.WriteLine(p);  
	}  
}
```