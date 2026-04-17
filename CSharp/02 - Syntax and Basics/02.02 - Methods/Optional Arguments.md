---
tags:
 - csharp
 - basics
 - methods
---

Optional arguments (also called optional parameters) allow you to define **default values** for method parameters. If the caller does not supply a value, the default is used.

```csharp
void CreateUser(string name, int age = 0, string city = "Unknown", bool isAdmin = false)
{
    Console.WriteLine($"{name}, {age}, {city}, Admin: {isAdmin}");
}

CreateUser("Alice");                        // age=0, city="Unknown", isAdmin=false
CreateUser("Bob", 25);                      // city="Unknown", isAdmin=false
CreateUser("Carol", 30, "Toronto");         // isAdmin=false
CreateUser("Dave", 40, "London", true);     // all specified
```

## Rules

- Optional parameters must appear **after all required parameters**
- Default values must be **compile-time constants** (literals, `const`, `default`, `enum` values)
- You cannot use `new` expressions or method calls as defaults

```csharp
// Valid
void Foo(int x, int y = 10, string s = "hello", bool flag = false) { }

// Invalid — new is not a compile-time constant
void Bar(int x, List<int> items = new List<int>()) { }  // compile error
```

## Skipping Optional Parameters with Named Arguments

Without [[Named Arguments]], you must supply every optional parameter in order up to the one you want to set:

```csharp
void Log(string message, string level = "INFO", bool timestamp = true, string source = "App")
{ ... }

// Want to set source but keep level and timestamp as defaults?
// Without named arguments — stuck passing all preceding values
Log("Hello", "INFO", true, "Database");

// With named arguments — skip straight to it
Log("Hello", source: "Database");
```

This is why named arguments and optional arguments work best together.

## Optional vs Overloading

Optional arguments can replace simple method overloads:

```csharp
// Overloading approach — 3 methods
void Log(string msg) { Log(msg, "INFO"); }
void Log(string msg, string level) { Log(msg, level, true); }
void Log(string msg, string level, bool timestamp) { ... }

// Optional arguments approach — 1 method
void Log(string msg, string level = "INFO", bool timestamp = true) { ... }
```

Optional arguments are simpler when the logic is the same. Overloads are better when each variant has **different behavior**.
