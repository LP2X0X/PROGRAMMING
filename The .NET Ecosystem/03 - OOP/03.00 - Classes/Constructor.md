---
tags:
 - csharp
 - oop
 - classes
---

- **Shortcut: ctor**
- Every C# class is provided with a “freebie” default constructor that you can redefine if need be. By definition, a default constructor never takes arguments. After allocating the new object into memory, the default constructor ensures that all field data of the class is set to an appropriate default value (see Chapter 3 for information regarding the default values of C# data types).
- However, as soon as you define a custom constructor with any number of parameters, the default constructor is silently removed from the class and is no longer available. Think of it this way: if you do not define a custom constructor, the C# compiler grants you a default to allow the object user to allocate an instance of your type with the field data set to the correct default values. However, when you define a unique constructor, the compiler assumes you have taken matters into your own hands. Therefore, if you want to allow the object user to create an instance of your type with the default constructor, as well as your custom constructor, you must explicitly redefine the default. To this end, understand that in a vast majority of cases, the implementation of the default constructor of a class is intentionally empty, as all you require is the ability to create an object with default values. 

--- 

## Chaining Constructor Calls Using this
In C#, constructor chaining allows one constructor to call another constructor in the same class using the `this` keyword. This can help reduce code duplication and improve code maintainability by reusing constructor logic.

### Constructor Chaining with `this`

When you have multiple constructors with different parameters, you can chain them to ensure that the initialization logic is centralized. Here’s how it works:

#### Example

Consider a class `Person` with multiple constructors:

```csharp
public class Person
{
    public string Name { get; }
    public int Age { get; }
    public string Address { get; }

    // Primary constructor with all parameters
    public Person(string name, int age, string address)
    {
        Name = name;
        Age = age;
        Address = address;
    }

    // Chained constructor with default address
    public Person(string name, int age) : this(name, age, "Unknown")
    {
    }

    // Chained constructor with default age and address
    public Person(string name) : this(name, 0, "Unknown")
    {
    }

    // Parameterless constructor
    public Person() : this("Unknown", 0, "Unknown")
    {
    }
}

class Program
{
    static void Main()
    {
        var person1 = new Person("Alice", 30, "123 Main St");
        var person2 = new Person("Bob", 25);
        var person3 = new Person("Charlie");
        var person4 = new Person();

        Console.WriteLine($"{person1.Name}, {person1.Age}, {person1.Address}");
        Console.WriteLine($"{person2.Name}, {person2.Age}, {person2.Address}");
        Console.WriteLine($"{person3.Name}, {person3.Age}, {person3.Address}");
        Console.WriteLine($"{person4.Name}, {person4.Age}, {person4.Address}");
    }
}
```

### Explanation

- **Primary Constructor**: The constructor with the most parameters (`name`, `age`, `address`) is considered the primary constructor. It contains the main initialization logic.
- **Chained Constructors**: Other constructors use the `this` keyword to call the primary constructor, passing default values for any missing parameters.
- **Parameterless Constructor**: This constructor calls the primary constructor with default values for all parameters.

### Benefits of Constructor Chaining

1. **Code Reuse**: Initialization logic is centralized in the primary constructor, reducing code duplication.
2. **Maintainability**: Changes to the initialization logic need to be made in only one place, making the code easier to maintain.
3. **Readability**: Constructors with fewer parameters delegate to more specific constructors, making the intent clearer.

### Advanced Example: Constructor Chaining with Validation

Constructor chaining can also include validation logic in the primary constructor:

```csharp
public class Person
{
    public string Name { get; }
    public int Age { get; }
    public string Address { get; }

    public Person(string name, int age, string address)
    {
        if (string.IsNullOrWhiteSpace(name))
            throw new ArgumentException("Name cannot be null or empty", nameof(name));

        if (age < 0)
            throw new ArgumentOutOfRangeException(nameof(age), "Age cannot be negative");

        Name = name;
        Age = age;
        Address = address;
    }

    public Person(string name, int age) : this(name, age, "Unknown")
    {
    }

    public Person(string name) : this(name, 0, "Unknown")
    {
    }

    public Person() : this("Unknown", 0, "Unknown")
    {
    }
}

class Program
{
    static void Main()
    {
        try
        {
            var person1 = new Person("Alice", 30, "123 Main St");
            var person2 = new Person("Bob", 25);
            var person3 = new Person("Charlie");
            var person4 = new Person();

            Console.WriteLine($"{person1.Name}, {person1.Age}, {person1.Address}");
            Console.WriteLine($"{person2.Name}, {person2.Age}, {person2.Address}");
            Console.WriteLine($"{person3.Name}, {person3.Age}, {person3.Address}");
            Console.WriteLine($"{person4.Name}, {person4.Age}, {person4.Address}");

            // This will throw an exception due to invalid age
            var invalidPerson = new Person("Dave", -1);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error: {ex.Message}");
        }
    }
}
```

---

## Private Constructors

A `private` constructor prevents external code from using `new` to create instances. This forces callers through a controlled creation path — typically a static factory method.

### Why Not Just Use a Public Constructor?

A public constructor **must return a new instance of its own type, or throw**. Those are the only two outcomes. A static factory method has more flexibility:

| Capability                            | Public Constructor | Static Factory |
| ------------------------------------- | ------------------ | -------------- |
| Validate and throw                    | Yes                | Yes            |
| Return `null` instead of throwing     | No                 | Yes            |
| Return a **cached/existing** instance | No                 | Yes            |
| Return a **subtype**                  | No                 | Yes            |
| Be `async`                            | No                 | Yes            |
| Have a meaningful **name**            | No                 | Yes            |

### Common Use Cases

#### Singleton — exactly one instance

```csharp
public class Singleton
{
    private static readonly Singleton _instance = new Singleton();
    public static Singleton Instance => _instance;

    private Singleton() { }
}
```

#### Factory method — controlled creation with return flexibility

```csharp
public class Connection
{
    private Connection(string connString) { /* ... */ }

    // Can return null instead of throwing
    public static Connection? TryCreate(string connString)
        => IsValid(connString) ? new Connection(connString) : null;

    // Can return a cached instance
    public static Connection GetOrCreate(string connString)
        => _cache.GetOrAdd(connString, cs => new Connection(cs));

    // Can return a subtype
    public static Connection Create(string connString)
        => connString.StartsWith("Server=")
            ? new SqlConnection(connString)
            : new SqliteConnection(connString);

    // Can be async
    public static async Task<Connection> CreateAsync(string connString)
    {
        var conn = new Connection(connString);
        await conn.InitializeAsync();
        return conn;
    }
}
```

#### Prevent inheritance — derived constructors must call a base constructor, so if all base constructors are private, no class can inherit from it

```csharp
public class Utility
{
    private Utility() { }

    public static void DoWork() => Console.WriteLine("Working");
}

// class MyUtility : Utility { }  // compile error — no accessible constructor
```

```ad-note
In modern C#, prefer `static class` over a private constructor for pure utility classes. A `static class` makes the intent explicit and also prevents instantiation.
```

---

## Static Constructors

- A **static constructor** (also called a type initializer) is a special constructor that initializes the **class itself**, not an instance. It runs exactly **once**, automatically, before the type is first used.
- It is used to initialize `static` fields, set up caches, load configuration, or perform any one-time class-level setup.

### Syntax

```csharp
class DatabaseManager
{
    private static string _connectionString;

    // Static constructor — no access modifier, no parameters
    static DatabaseManager()
    {
        _connectionString = LoadFromConfig();
        Console.WriteLine("Static constructor called — class initialized.");
    }

    private static string LoadFromConfig() => "Server=localhost;Database=mydb;";

    public void Connect()
        => Console.WriteLine($"Connecting with: {_connectionString}");
}
```

### Rules

| Rule | Detail |
|---|---|
| No access modifier | You cannot write `public static DatabaseManager()` — it is always implicitly `private` |
| No parameters | A static constructor takes zero arguments — there is no way to call it manually |
| Runs exactly once | The CLR guarantees it runs only once per type, per application domain |
| Cannot be called directly | The runtime calls it automatically — you never write `ClassName.StaticConstructor()` |
| Runs before first use | Triggered before the first instance is created OR before any static member is accessed |
| Thread-safe by default | The CLR ensures only one thread executes the static constructor, even in multithreaded scenarios |

### When Does It Run?

- The CLR calls the static constructor at some point **before** the type is first accessed. The exact timing is:
  1. Before the first instance of the class is created, OR
  2. Before any static member (field, property, method) is referenced

```csharp
class Example
{
    public static int Counter;

    static Example()
    {
        Counter = 42;
        Console.WriteLine("Static ctor ran");
    }

    public Example()
    {
        Console.WriteLine("Instance ctor ran");
    }
}

// Accessing a static member triggers the static constructor:
Console.WriteLine(Example.Counter);
// Output:
//   Static ctor ran
//   42

// Creating an instance — static ctor already ran, so only instance ctor fires:
var e = new Example();
// Output:
//   Instance ctor ran
```

### Static Constructor vs Static Field Initializer

- You can initialize static fields inline without a static constructor. The compiler generates a static constructor behind the scenes.

```csharp
// These two are functionally equivalent:

// Option 1: Inline initializer (compiler generates static ctor)
class Config
{
    private static readonly string _env = Environment.GetEnvironmentVariable("ENV") ?? "dev";
}

// Option 2: Explicit static constructor
class Config
{
    private static readonly string _env;

    static Config()
    {
        _env = Environment.GetEnvironmentVariable("ENV") ?? "dev";
    }
}
```

- Use an explicit static constructor when initialization involves **multiple steps**, **error handling**, or **logic that can't fit in a single expression**.

### Error Handling

```ad-warning
If a static constructor throws an exception, the type becomes **permanently unusable** for the rest of the application's lifetime. Any subsequent attempt to use the type will throw a `TypeInitializationException`. There is no retry mechanism — the static constructor will NOT run again.
```

```csharp
class Fragile
{
    static Fragile()
    {
        throw new Exception("Init failed");
    }
}

try { var f = new Fragile(); }
catch (TypeInitializationException ex)
{
    // ex.InnerException is the original "Init failed" exception
    Console.WriteLine(ex.InnerException.Message);
}

// Every future attempt also throws TypeInitializationException — the type is dead
```

Because of this, keep static constructors **simple and unlikely to fail**. Avoid network calls, file I/O, or anything that could throw intermittently.

### Practical Example: Singleton Pattern

- Static constructors are commonly used to implement a thread-safe singleton without explicit locking:

```csharp
class Singleton
{
    private static readonly Singleton _instance;

    static Singleton()
    {
        _instance = new Singleton();
    }

    private Singleton() { }

    public static Singleton Instance => _instance;
}

// Thread-safe: the CLR guarantees the static constructor runs
// exactly once, even if multiple threads access Instance simultaneously
```

### Static Constructor + Instance Constructor

- A class can have **both**. They serve different purposes and run at different times.

```csharp
class Logger
{
    private static readonly string _logPath;
    private readonly string _source;

    // Runs once — initializes class-level state
    static Logger()
    {
        _logPath = Path.Combine(AppContext.BaseDirectory, "app.log");
    }

    // Runs every time — initializes instance-level state
    public Logger(string source)
    {
        _source = source;
    }

    public void Log(string message)
        => File.AppendAllText(_logPath, $"[{_source}] {message}\n");
}
```

---

## Default Constructor and Object Initializers

When you define **any** custom constructor, the compiler removes the implicit parameterless constructor. This affects object initializer syntax (`new MyClass { ... }`), which requires a parameterless constructor (or an explicit constructor call).

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }

    public Person(string name)
    {
        Name = name;
    }
}

// Error — no parameterless constructor exists anymore
var p = new Person { Name = "Long", Age = 25 };
```

### Fixes

```csharp
// Option 1: Add back the parameterless constructor explicitly
public Person() { }
public Person(string name) { Name = name; }

var p = new Person { Name = "Long", Age = 25 };

// Option 2: Use the parameterized constructor together with the initializer
var p = new Person("Long") { Age = 25 };
```

```ad-note
Object initializer syntax (`{ Property = value }`) is syntactic sugar — the compiler calls the constructor first, then sets each property. So a parameterless constructor (or an explicit constructor call) is always required.
```

---

## Summary

### Constructor Chaining

Constructor chaining in C# using the `this` keyword is a powerful technique to improve code reuse and maintainability. By delegating initialization logic to a primary constructor, you can ensure that all constructors in your class share common logic, making your code cleaner and more consistent.

### Private Constructors

| Use Case            | Why Private Constructor                                            |
| ------------------- | ------------------------------------------------------------------ |
| Singleton           | Exactly one instance, globally accessible                          |
| Factory pattern     | Full control over creation: caching, subtypes, null returns, async |
| Builder pattern     | Only the inner `Builder` class creates instances                   |
| Prevent inheritance | No external class can derive from it                               |
| Static-only class   | Prevent meaningless instantiation (prefer `static class`)          |

The private constructor is the lock on the door — the factory method is the key you hand out on your terms.

### Static vs Instance Constructors

| | Instance Constructor | Static Constructor |
|---|---|---|
| Initializes | instance fields | static fields / class-level state |
| Runs | every time `new` is called | exactly once, before first use |
| Parameters | yes | no |
| Access modifier | any (`public`, `private`, etc.) | none (implicitly private) |
| Can be overloaded | yes | no (only one allowed) |
| Called by | your code via `new` | the CLR automatically |