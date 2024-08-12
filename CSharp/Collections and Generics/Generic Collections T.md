Generics in C# provide a way to define classes, interfaces, and methods with placeholders for the types they operate on. This allows for code reusability, type safety, and performance benefits. Here are the key concepts and features of generics in C#.

```ad-note
Only classes, structures, interfaces, and delegates can be written generically; enum types cannot.
```

Formally speaking, you call these tokens **type parameters**; however, in more user-friendly terms, you can simply call them **placeholders**. You can read the symbol <T\> as “of T.” Thus, you can read IEnumerable<T\> as “IEnumerable of T” or, to say it another way, “IEnumerable of type T.

```ad-important
The name of a type parameter (placeholder) is irrelevant, and it is up to the developer who created the generic item. However, typically T is used to represent types, TKey or K is used for keys, and TValue or V is used for values.
```

```ad-tip
Even though the compiler can discover the correct type parameter based on the data type used to declare your parameters, you should get in the habit of always specifying the type parameter explicitly.
````csharp
SwapFunctions.Swap<bool>(ref b1, ref b2);
```

### Defining Generics

1. **Generic Classes**:
    ```csharp
    public class GenericClass<T>
    {
        private T _value;
        
        public GenericClass(T value)
        {
            _value = value;
        }

        public T GetValue()
        {
            return _value;
        }
    }
    ```

2. **Generic Methods**:
    ```csharp
    public class MyClass
    {
        public T GenericMethod<T>(T param)
        {
            return param;
        }
    }
    ```

3. **Generic Interfaces**:
    ```csharp
    public interface IGenericInterface<T>
    {
        T Get();
        void Set(T value);
    }
    ```

4. **Generic Delegates**:
    ```csharp
    public delegate T MyGenericDelegate<T>(T param);
    ```

### Using Generics

1. **Instantiating Generic Classes**:
    ```csharp
    var genericInt = new GenericClass<int>(10);
    int value = genericInt.GetValue(); // value is 10

    var genericString = new GenericClass<string>("Hello");
    string str = genericString.GetValue(); // str is "Hello"
    ```

2. **Calling Generic Methods**:
    ```csharp
    var myClass = new MyClass();
    int intValue = myClass.GenericMethod<int>(10);
    string stringValue = myClass.GenericMethod<string>("Hello");
    ```

3. **Implementing Generic Interfaces**:
    ```csharp
    public class GenericInterfaceImpl<T> : IGenericInterface<T>
    {
        private T _value;

        public T Get()
        {
            return _value;
        }

        public void Set(T value)
        {
            _value = value;
        }
    }
    ```

### Constraints on Generics

Constraints can be applied to restrict the types that can be used as arguments for type parameters:

1. **where T : struct** (value type constraint):
    ```csharp
    public class GenericClass<T> where T : struct
    {
        // T must be a value type
    }
    ```

2. **where T : class** (reference type constraint):
    ```csharp
    public class GenericClass<T> where T : class
    {
        // T must be a reference type
    }
    ```

3. **where T : new()** (parameterless constructor constraint):
    ```csharp
    public class GenericClass<T> where T : new()
    {
        public T CreateInstance()
        {
            return new T(); // T must have a parameterless constructor
        }
    }
    ```

4. **where T : SomeBaseClass** (base class constraint):
    ```csharp
    public class GenericClass<T> where T : SomeBaseClass
    {
        // T must inherit from SomeBaseClass
    }
    ```

5. **where T : ISomeInterface** (interface constraint):
    ```csharp
    public class GenericClass<T> where T : ISomeInterface
    {
        // T must implement ISomeInterface
    }
    ```

### Generic Collections

The .NET Framework provides several generic collections in the `System.Collections.Generic` namespace:

1. **List<T>**:
    ```csharp
    List<int> intList = new List<int>();
    intList.Add(1);
    intList.Add(2);
    ```

2. **Dictionary<TKey, TValue>**:
    ```csharp
    Dictionary<int, string> dict = new Dictionary<int, string>();
    dict.Add(1, "one");
    dict.Add(2, "two");
    ```

3. **Queue<T>**:
    ```csharp
    Queue<string> queue = new Queue<string>();
    queue.Enqueue("first");
    queue.Enqueue("second");
    ```

4. **Stack<T>**:
    ```csharp
    Stack<double> stack = new Stack<double>();
    stack.Push(1.1);
    stack.Push(2.2);
    ```

Generics in C# provide powerful tools for creating reusable, type-safe code that performs well. By understanding and using generics effectively, you can significantly improve the flexibility and maintainability of your applications.