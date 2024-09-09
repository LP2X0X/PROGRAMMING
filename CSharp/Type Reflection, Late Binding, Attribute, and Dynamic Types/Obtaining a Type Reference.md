### Obtaining a Type Reference Using System.Object.GetType()  
- You can obtain an instance of the Type class in a variety of ways. However, the one thing you cannot do is directly create a Type object using the new keyword, as Type is an abstract class. System.Object defines a method named GetType(), which returns an instance of the Type class that represents the metadata for the current object.  
```csharp
// Obtain type information using a SportsCar instance.  
SportsCar sc = new SportsCar();  
Type t = sc.GetType();  
```
- Obviously, this approach will work only if you have compile-time knowledge of the type you want to reflect over (SportsCar in this case) and currently have an instance of the type in memory. Given this restriction, it should make sense that tools such as ildasm.exe do not obtain type information by directly calling System.Object.GetType() for each type since that ildasm.exe was not compiled against your custom assemblies.

### Obtaining a Type Reference Using typeof()
- The next way to obtain type information is using the C# typeof operator, like so:  
```csharp
// Get the type using typeof.  
Type t = typeof(SportsCar);  
```
- Unlike System.Object.GetType(), the typeof operator is helpful, in that you do not need to first create an object instance to extract type information. However, your code base must still have compile time knowledge of the type you are interested in examining, as typeof expects the strongly typed name of the type.

### Obtaining a Type Reference Using System.Type.GetType()
- To obtain type information in a more flexible manner, you may call the static GetType() member of the System.Type class and specify the fully qualified string name of the type you are interested in examining. Using this approach, you do not need to have compile-time knowledge of the type you are extracting metadata from, given that Type.GetType() takes an instance of the omnipresent System.String.
- The Type.GetType() method has been overloaded to allow you to specify two Boolean parameters, one of which controls whether an exception should be thrown if the type cannot be found, and the other of which establishes the case sensitivity of the string.

```csharp
// Obtain type information using the static Type.GetType() method  
// (don't throw an exception if SportsCar cannot be found and ignore case).  
Type t = Type.GetType("CarLibrary.SportsCar", false, true);
```
- In the previous example, notice that the string you are passing into GetType() makes no mention of the assembly containing the type. In this case, the assumption is that the type is defined within the currently executing assembly. However, when you want to obtain metadata for a type within an external assembly, the string parameter is formatted using the type’s fully qualified name, followed by a comma, followed by the friendly name (the assembly name without any version information) of the assembly containing the type, like so:
```csharp
// Obtain type information for a type within an external assembly.  
Type t = Type.GetType("CarLibrary.SportsCar, CarLibrary");
```
- As well, do know that the string passed into Type.GetType() may specify a plus token (+) to denote a nested type. Assume you want to obtain type information for an enumeration (SpyOptions) nested within a class named JamesBondCar.
```csharp
// Obtain type information for a nested enumeration  
// within the current assembly.  
Type t = Type.GetType("CarLibrary.JamesBondCar+SpyOptions");
```