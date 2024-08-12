The answer: a copy of the reference to the caller’s object.  
Therefore, as the method is pointing to the same object as the caller, it is possible to alter the object’s state data. What is not possible is to reassign what the reference is **pointing** to.

```csharp
static void SendAPersonByValue(Person p)  
{  
	// Change the age of "p"?  
	p.personAge = 99;  // This will work
	// Will the caller see this reassignment?  
	p = new Person("Nikki", 99); // This will not work 
}
```