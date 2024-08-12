The C# using keyword also lets you create an alias for a type’s fully qualified name. When you do so, you define a token that is substituted for the type’s full name at compile time. Defining aliases provides a second way to resolve name clashes.

### Example
```csharp
using Rppr = Namespace.To.RelocatedPlantProductReleaseItem;
```