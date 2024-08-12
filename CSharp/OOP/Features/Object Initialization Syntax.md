- The object initializers syntax allows you to create an instance, and after that it assigns the newly created object, with its assigned properties, to the variable in the assignment.
```csharp
Cat cat = new Cat { Age = 10, Name = "Fluffy" };
```

```ad-note
it’s important to remember that the object initialization process is using the property setter implicitly. If the property setter is marked private, this syntax cannot be used.
```

```ad-note
Behind the scenes, the type’s default constructor is invoked, followed by setting the values to the specified properties.
```