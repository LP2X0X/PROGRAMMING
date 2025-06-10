🟩 1. Basic Syntax & Structure
- Namespace: Container for classes and other types.
- Class: Blueprint to create objects.
- Main Method: Entry point of a C# program — static void Main(string[] args)
- Statements end with a semicolon ;.
- Case-sensitive language.

🟩 2. Data Types
- Value Types: int, float, double, decimal, bool, char, struct, enum
- Reference Types: object, string, dynamic, arrays, class, interface, delegate
- Nullable Types: int?, bool?, etc.

🟩 3. Variables & Constants
- Variables: Declared with type → int x = 10;
- Implicit typing: var keyword → var name = "John";
- Constants: const int x = 100; (compile-time)
- Readonly: readonly int y; (runtime set, only once)

🟩 4. Type Conversions
- Implicit: No data loss (int to float)
- Explicit: Cast required (float to int)
- Convert class: Convert.ToInt32(), etc.
- Parsing: int.Parse(), int.TryParse()

🟩 5. Operators
- Arithmetic: +, -, *, /, %
- Relational: ==, !=, >, <, >=, <=
- Logical: &&, ||, !
- Assignment: =, +=, -=
- Null-coalescing: ??, ??=
- Ternary: condition ? trueExpr : falseExpr

🟩 6. Control Flow
- if, else if, else
- switch
- for, while, do-while
- foreach (used with collections)
- break, continue, return, goto

🟩 7. Object-Oriented Programming (OOP)
- Class and Object: Blueprint and instance
- Encapsulation: Using access modifiers (public, private, protected, internal)
- Inheritance: class Car : Vehicle
- Polymorphism:
  - Method Overloading: Same method name, different parameters
  - Method Overriding: virtual & override
- Abstraction: Abstract classes and interfaces

🟩 8. Access Modifiers
- public
- private
- protected
- internal
- protected internal
- private protected

🟩 9. Properties
- Encapsulates fields with get/set:
  public int Age { get; set; }
- Readonly/Writeonly: set/get-only property
- Expression-bodied properties: public int Age => _age;

🟩 10. Constructors & Destructors
- Constructor: Same name as class, initializes object
- Overloaded constructors
- Static constructors: Initializes static members
- Destructor: ~ClassName() — cleans resources (less used)

🟩 11. Static Members
- Belong to class instead of instance
- static fields, methods, classes

🟩 12. Interfaces & Abstract Classes
- Interface: Syntax contract, no implementation
  interface IDrive { void Drive(); }
- Abstract Class: Partial implementation
  abstract class Vehicle { abstract void Drive(); }

🟩 13. Collections
- Arrays: int[] arr = new int[5];
- Lists: List<int> list = new List<int>();
- Dictionary<TKey, TValue>
- Queue, Stack, HashSet, LinkedList, etc.

🟩 14. Exception Handling
- try { } catch(Exception ex) { } finally { }
- throw new Exception("message");
- Custom exceptions

🟩 15. Delegates & Events
- Delegate: Type-safe function pointer
  delegate int MathOperation(int x, int y);
- Multicast delegates
- Events: Publisher-subscriber model

🟩 16. Lambda Expressions & LINQ
- Lambda: A => A + 1
- LINQ (Language Integrated Query):
  var result = from e in list where e.Age > 30 select e;
  or
  var result = list.Where(e => e.Age > 30).ToList();

🟩 17. Asynchronous Programming
- async / await
- Task and Task<T>
- Threading, Parallel programming

🟩 18. File I/O
- System.IO namespace
- File, FileInfo, Directory, StreamReader, StreamWriter, etc.

🟩 19. Generics
- Generic class/method: List<T>, Dictionary<TKey, TValue>
- Benefits: Type safety, reusability
- Constraints: where T : class, new(), etc.

🟩 20. Attributes & Reflection
- Attribute: Metadata attached to code
  [Obsolete], [Serializable], [TestMethod], etc.
- Reflection: Reading metadata at runtime from System.Reflection

🟩 21. Namespaces & Assemblies
- Namespace: Organize code
- Assembly: .dll or .exe — compiled output
- using directive: Include namespaces

🟩 22. Structs & Enums
- Structs: Value types — struct Point { public int x, y; }
- Enums: enum Days { Sunday, Monday }

🟩 23. Nullable Types & Null Handling
- Nullable types: int?
- Null coalescing: var name = value ?? "Default";
- Null-conditional operator: obj?.Method()

🟩 24. Pattern Matching
- switch & is patterns:
  if (obj is string s)
  switch(obj)
  {
    case int i when i > 0: break;
  }

🟩 25. Records (C# 9+)
- Immutable data types
- record Person(string Name, int Age);

🟩 26. Tuples
- Lightweight grouping: (int, string) tuple = (1, "A");
- Deconstruction: var (x, y) = tuple;

🟩 27. Dynamic Type
- Type checked at runtime
- dynamic obj = GetSomeObject();

🟩 28. Unsafe Code & Pointers (Advanced)
- unsafe keyword allows pointer operations
- Requires /unsafe compiler option

🟩 29. Dependency Injection (Common in .NET Core)
- Registering and injecting dependencies (services)
- Scoped, Singleton, Transient lifetimes

🟩 30. C# Versions & Features
- C# is continuously evolving: each version introduces features.
  - C# 7.x: Tuples, pattern matching, out variables
  - C# 8.0: Nullable reference types, switch expressions
  - C# 9.0: Records, init-only setters
  - C# 10/11: File-scoped namespaces, global using, etc.
