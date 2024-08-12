In C# and .NET, there are several built-in attributes provided by the framework that you can apply to classes, methods, properties, fields, and other code elements. These attributes serve various purposes and provide metadata that can influence compilation, runtime behavior, serialization, and more. Here are some commonly used built-in attributes:

1. **`[Obsolete]`**: Marks a program entity, such as a class or method, as obsolete and provides a message to the compiler or consumer indicating it should no longer be used. Example:
   ```csharp
   [Obsolete("This method is obsolete, use NewMethod instead.")]
   public void OldMethod() { ... }
   ```

2. **`[Serializable]`**: Indicates that a class can be serialized. This is used for classes that you intend to serialize to a binary format or XML. Example:
   ```csharp
   [Serializable]
   public class MyClass { ... }
   ```

3. **`[DllImport]`**: Specifies that the attributed method is exposed by an unmanaged DLL and provides information about how to call the method. Example:
   ```csharp
   [DllImport("user32.dll")]
   public static extern int MessageBox(IntPtr hWnd, string text, string caption, uint type);
   ```

4. **`[Conditional]`**: Specifies that a method call or attribute should be conditional based on a compilation symbol (like `DEBUG`). Example:
   ```csharp
   [Conditional("DEBUG")]
   public void DebugMethod() { ... }
   ```

5. **`[ThreadStatic]`**: Indicates that the value of a static field is unique to each thread. Example:
   ```csharp
   [ThreadStatic]
   public static int ThreadLocalValue;
   ```

6. **`[DefaultValue]`**: Specifies the default value for a property or field, typically used by designers and serializers. Example:
   ```csharp
   [DefaultValue(10)]
   public int DefaultPropertyValue { get; set; }
   ```

7. **`[DataContract]` and `[DataMember]`**: Used in WCF (Windows Communication Foundation) to specify that a class or member should be included in data contracts for serialization. Example:
   ```csharp
   [DataContract]
   public class Person
   {
       [DataMember]
       public string Name { get; set; }
       [DataMember]
       public int Age { get; set; }
   }
   ```

8. **`[Browsable]`**: Specifies whether a property or field should be displayed in visual designers. Example:
   ```csharp
   [Browsable(true)]
   public int DisplayedProperty { get; set; }
   ```

These are just a few examples of built-in attributes available in C# and .NET. Attributes provide a powerful way to annotate and provide metadata to your code, influencing how it is compiled, serialized, and interacted with by various tools and frameworks.