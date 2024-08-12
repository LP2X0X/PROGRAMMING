Implementing Dependency Injection (DI) without using a DI container involves manually providing dependencies to the classes that need them. This can be done through constructor injection, method injection, or property injection. Let's look at an example using constructor injection:

### Example Without DI Container:

```csharp
// Logger interface
public interface ILogger
{
    void Log(string message);
}

// Concrete implementation of ILogger
public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}

// Class using dependency injection without a container
public class DIInjectedService
{
    private readonly ILogger _logger;

    // Constructor injection
    public DIInjectedService(ILogger logger)
    {
        _logger = logger;
    }

    public void DoSomething()
    {
        _logger.Log("Doing something");
        // Business logic here
    }
}

// Usage of DIInjectedService without a DI container
class Program
{
    static void Main()
    {
        // Manually create instances of dependencies
        ILogger logger = new ConsoleLogger();
        DIInjectedService service = new DIInjectedService(logger);

        // Use the service
        service.DoSomething();
    }
}
```

In this example:

- The `DIInjectedService` class depends on an `ILogger`.
- Constructor injection is used to provide the `ILogger` dependency. The dependency is passed to the class through its constructor.
- The `Main` method manually creates instances of the dependencies (`ConsoleLogger` and `DIInjectedService`) and injects the logger into the service.

### Benefits:

1. **Decoupling:**
   - The dependencies are injected rather than created inside the dependent class, promoting decoupling.
  
2. **Testability:**
   - The class is more testable since you can easily substitute dependencies with mock objects during testing.

3. **Flexibility:**
   - You can easily switch or update implementations of dependencies without modifying the dependent class.

### Drawbacks:

1. **Manual Management:**
   - Dependencies need to be manually managed and injected, which can lead to boilerplate code in larger applications.

2. **Complex Graphs:**
   - For complex object graphs, managing dependencies manually might become cumbersome.

3. **Initialization Order:**
   - Dependencies need to be provided in the correct order during object creation, which can be error-prone.

While using a DI container can automate the process of dependency resolution and management, the manual approach is a valid choice, especially in smaller projects or scenarios where the benefits of a container might not justify its introduction.