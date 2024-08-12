- Using callbacks, programmers were able to configure one function to report back to (call back) another function in the application. With this approach, Windows developers were able to handle button clicking, mouse moving, menu selecting, and general bidirectional communications between two entities in memory.
- In the .NET and .NET Core (and .NET 5+) Frameworks, callbacks are accomplished in a type-safe and object-oriented manner using delegates. A delegate is a type-safe object that points to another method (or possibly a list of methods) in the application, which can be invoked later. Specifically, a delegate maintains three important pieces of information.
	- The address of the method on which it makes calls
	- The parameters (if any) of this method
	- The return type (if any) of this method

```ad-note
.NET delegates can point to either static or instance methods.
```

---

## Defining a Delegate Type in C\#
- When you want to create a delegate type in C#, you use the delegate keyword. The name of your delegate type can be whatever you desire. However, you must define the delegate to match the signature of the method(s) it will point to. For example, the following delegate type (named BinaryOp) can point to any method that returns an integer and takes two integers as input parameters:

```csharp
// This delegate can point to any method,  
// taking two integers and returning an integer.  
public delegate int BinaryOp(int x, int y);
```
- As you can see, the compiler-generated BinaryOp class defines three public methods. Invoke() is the key method in .NET, as it is used to invoke each method maintained by the delegate object in a synchronous manner, meaning the caller must wait for the call to complete before continuing its way.
- To summarize, a C# delegate type definition results in a sealed class with a compiler-generated method whose parameter and return types are based on the delegate’s declaration.