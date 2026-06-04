---
tags:
 - csharp
 - reflection
---

## Obtaining Type Information

Four ways to get a `Type` object:

```csharp
// 1. typeof -- compile-time, no instance needed
Type t1 = typeof(Person);

// 2. GetType() -- on an existing instance
Person p = new Person();
Type t2 = p.GetType();

// 3. Type.GetType() -- from a string name (assembly-qualified for external types)
Type t3 = Type.GetType("MyApp.Person");
Type t4 = Type.GetType("System.Int32, System.Private.CoreLib");

// 4. Assembly.GetType() -- from a loaded assembly
Assembly asm = Assembly.GetExecutingAssembly();
Type t5 = asm.GetType("MyApp.Person");
```

| Method | Needs instance? | Compile-time safe? | Use when |
| --- | --- | --- | --- |
| `typeof(T)` | No | Yes | Type known at compile time |
| `obj.GetType()` | Yes | Yes | You have an instance |
| `Type.GetType("name")` | No | No (string) | Type name as a string |
| `assembly.GetType("name")` | No | No (string) | Type from a specific assembly |

## Inspecting Members

```csharp
Type t = typeof(Person);

MethodInfo[] methods = t.GetMethods();        // public methods
PropertyInfo[] props = t.GetProperties();     // public properties
FieldInfo[] fields = t.GetFields();           // public fields
ConstructorInfo[] ctors = t.GetConstructors(); // public constructors
EventInfo[] events = t.GetEvents();           // public events
```

Each returns **public members by default**. Use `BindingFlags` for more control.

## Using BindingFlags

`BindingFlags` is a bitmask enum that controls which members are returned. You combine flags with `|`:

```csharp
// Get ALL members -- public + private, instance + static
FieldInfo[] allFields = t.GetFields(
    BindingFlags.Public | BindingFlags.NonPublic |
    BindingFlags.Instance | BindingFlags.Static);
```

| Flag | Meaning |
| --- | --- |
| `Public` | Include public members |
| `NonPublic` | Include private/protected/internal members |
| `Instance` | Include instance members |
| `Static` | Include static members |
| `DeclaredOnly` | Exclude inherited members |
| `FlattenHierarchy` | Include static members from base types |

```ad-important
You **must** specify at least one of `Instance` or `Static`, **AND** at least one of `Public` or `NonPublic` -- otherwise you get an empty array back. This is the most common mistake when using `BindingFlags`.
```

## Creating Instances Dynamically

`Activator.CreateInstance` calls the constructor matching the supplied arguments:

```csharp
// Default (parameterless) constructor
object obj = Activator.CreateInstance(typeof(Person));

// Constructor with parameters -- matched by type
object obj2 = Activator.CreateInstance(typeof(Person), "Long", 28);
```

If the type has no matching constructor, you get a `MissingMethodException`.

## Reading and Writing Fields/Properties

```csharp
Type t = typeof(Person);
Person p = new Person { Name = "Long", Age = 28 };

// Read a property
PropertyInfo prop = t.GetProperty("Name");
string name = (string)prop.GetValue(p);  // "Long"

// Write a property
prop.SetValue(p, "Pham");

// Read a private field -- requires BindingFlags
FieldInfo field = t.GetField("_id",
    BindingFlags.NonPublic | BindingFlags.Instance);
int id = (int)field.GetValue(p);
```

## Invoking Methods

```csharp
Type t = typeof(Person);
Person p = new Person();

MethodInfo greet = t.GetMethod("Greet");

// First arg = target instance (null for static methods)
// Second arg = method parameters as object[]
object result = greet.Invoke(p, new object[] { "Hello" });
```

## Accessing Private Members

```csharp
Type t = typeof(Person);
Person p = new Person();

// Private field
FieldInfo privateField = t.GetField("_secret",
    BindingFlags.NonPublic | BindingFlags.Instance);
object val = privateField.GetValue(p);

// Private method
MethodInfo privateMethod = t.GetMethod("InternalCalc",
    BindingFlags.NonPublic | BindingFlags.Instance);
privateMethod.Invoke(p, null);
```

```ad-warning
Accessing private members **breaks encapsulation**. Use this sparingly -- mainly for testing, serialization frameworks, or debugging. Private APIs can change without notice between versions, so code that depends on them is fragile.
```

## Performance Considerations

Reflection is significantly slower than direct calls. The costs come from:

- **Boxing/unboxing** -- `Invoke` takes `object[]`, so value types get boxed on every call
- **Security checks** -- access permissions are verified on every invocation
- **No JIT optimization** -- reflected calls cannot be inlined or devirtualized
- **String-based lookup** -- `GetMethod("Name")` scans the method table each time

For hot paths, **cache** the `MethodInfo`/`PropertyInfo` object and consider using compiled delegates (`Delegate.CreateDelegate` or expression trees) or source generators instead of raw reflection.

## See Also

- [[Reflection Overview]]
- [[Reflecting on Static Types]]
- [[Reflecting on Generic Types]]
- [[Dynamically Loading Assemblies]]
- [[The System.Type Class]]
