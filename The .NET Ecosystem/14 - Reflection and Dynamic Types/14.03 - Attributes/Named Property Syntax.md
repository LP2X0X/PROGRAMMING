---
tags:
 - csharp
 - attributes
---

## Named Property Syntax

When applying an attribute, you can set values on its public writable properties using **named property syntax**. This is how you pass optional metadata beyond the constructor parameters.

```csharp
// VehicleDescriptionAttribute has a constructor param AND a writable property
public class VehicleDescriptionAttribute : Attribute
{
    public string Name { get; }              // set via constructor (positional)
    public string Description { get; set; }  // set via named property (optional)

    public VehicleDescriptionAttribute(string name)
    {
        Name = name;
    }
}

// Apply with named property
[VehicleDescription("Harley", Description = "My rocking Harley")]
public class Motorcycle { }

// Description is optional -- you can omit it
[VehicleDescription("Honda")]
public class Car { }
```

## Positional vs Named -- When to Use

| | Positional (constructor) | Named (property) |
|---|---|---|
| Defined by | Constructor parameter | Public writable property or field |
| Required? | Yes -- must provide | No -- optional |
| Use for | Identity / required data | Optional configuration |

```ad-note
Named property syntax is only legal if the attribute class has a **writable** .NET property or public field. Read-only properties cannot be set this way -- they must be set through the constructor as positional arguments.
```

## Multiple Named Properties

You can have as many named properties as you need. They can appear in any order after the positional arguments:

```csharp
[AttributeUsage(AttributeTargets.Class)]
public class ApiEndpointAttribute : Attribute
{
    public string Route { get; set; }
    public string Method { get; set; } = "GET";    // default value
    public bool RequiresAuth { get; set; }
}

// All named -- no constructor params in this case
[ApiEndpoint(Route = "/users", Method = "POST", RequiresAuth = true)]
public class CreateUserHandler { }

// Only set what you need -- Method defaults to "GET", RequiresAuth to false
[ApiEndpoint(Route = "/health")]
public class HealthCheckHandler { }
```

## See Also

- [[Custom Attributes]]
- [[AttributeUsage]]
- [[Attribute Overview]]
