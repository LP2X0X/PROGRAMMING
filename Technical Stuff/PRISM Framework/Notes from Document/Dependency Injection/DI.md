## Definition
- Dependency Injection allows components to obtain references to the other components that they depend on without having to hard code those references, thereby promoting better code re-use and improved flexibility.
![[aa.jpeg|center|500]]
![[ab.png|center]]
## Types
1. **Constructor Injection:** When the Injector Injects the Dependency Object (i.e. Service Object) into the Client Class through the Client Class Constructor, then it is called Constructor Dependency Injection.
2. **Property Injection:** When the Injector Injects the Dependency Object (i.e. Service Object) into the Client Class through the public Property of the Client Class, then it is called Property Dependency Injection. This is also called the Setter Injection.
3. **Method Injection:** When the Injector Injects the Dependency Object (i.e. Service Object) into the Client Class through a public Method of the Client Class, then it is called Method Dependency Injection.
## Implementation
```csharp
public class DIInjectedService
{
    private readonly ILogger _logger;

    public DIInjectedService(ILogger logger)
    {
        _logger = logger; // Inject through constructor
    }

    public void DoSomething()
    {
        _logger.Log("Doing something");
        // Business logic here
    }
}
```
## Benefits
1. **Decoupling:**
    
    - DI allows for better decoupling between components. In the refactored code, `DIInjectedService` is no longer responsible for creating its dependencies.
    - This reduces the class's responsibilities, making it more focused on its primary task.
2. **Testability:**
    
    - Dependency Injection simplifies unit testing. With DI, you can easily provide mock or stub implementations of dependencies during testing.
    - In the original code, testing might involve modifying or extending the class to allow for substitution of dependencies.
3. **Flexibility:**
    
    - DI enables greater flexibility in switching or updating implementations. You can easily replace one implementation of `ILogger` with another without modifying the `DIInjectedService` class.
    - In the original code, changing dependencies would require modifying the class directly, potentially causing ripple effects.
4. **Maintainability:**
    
    - Code maintainability is improved because changes to dependencies don't affect the dependent class. This adheres to the Open/Closed Principle (OCP) of SOLID.
    - The original code is more rigid, making it harder to accommodate changes without modifying the class.
5. **Reusability:**
    
    - Components become more reusable as they are not tightly coupled to concrete implementations. You can use the same `DIInjectedService` with different implementations of `ILogger` across various parts of the application.
    - The original code is less reusable, as each class creates and manages its own dependencies.
6. **Inversion of Control (IoC):**
    
    - DI promotes the Inversion of Control (IoC) principle, where the control over the creation and management of dependencies is shifted to an external entity (DI container).
    - The original code represents a form of "Control Flow" where classes have control over their dependencies.