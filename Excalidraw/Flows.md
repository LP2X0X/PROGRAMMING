## **1 - Send data to LCS**
![[Pasted image 20241016101055.png|center]]
- Send for FOV - ARRAY - BOARD Inspection Completion
![[Pasted image 20241016101228.png|center]]
## **2 - Options of Badmark**
![[Pasted image 20241016100614.png|center]]
- Why we can not stop here using old technique?
![[image.png|center]]
- *Remember*: Topics in evt in EventNetwork which help send event to correct object
![[Pasted image 20241016095513.png|500|center]]
## **3 - Union attribute of  MsgPack**
In **MessagePack for C#** (.NET), the concept of a "union attribute" is used to handle polymorphism, allowing serialization and deserialization of different derived types within a single base type. This feature is crucial when working with interfaces or abstract classes where multiple implementations need to be serialized or deserialized.

### Using `[Union]` Attribute

To define a union in MessagePack for C#, you annotate a base class or interface with the `[Union]` attribute, specifying different types that could be used during serialization and deserialization. Each derived type is given a unique identifier. Here's how you can use it:

1. **Base Type Definition with `[MessagePackObject]`**:
   The base class or interface should be decorated with `[MessagePackObject]` if it's a class. You then apply `[Union]` attributes for each derived type.

2. **Derived Types Definition**:
   The derived types must also be annotated with `[MessagePackObject]`.

### Example

```csharp
using MessagePack;

[MessagePackObject]
[Union(0, typeof(Cat))]
[Union(1, typeof(Dog))]
public interface IAnimal
{
    [Key(0)]
    string Name { get; set; }
}

[MessagePackObject]
public class Cat : IAnimal
{
    [Key(0)]
    public string Name { get; set; }

    [Key(1)]
    public int Lives { get; set; }
}

[MessagePackObject]
public class Dog : IAnimal
{
    [Key(0)]
    public string Name { get; set; }

    [Key(1)]
    public string Breed { get; set; }
}

class Program
{
    static void Main()
    {
        IAnimal animal = new Cat { Name = "Whiskers", Lives = 9 };

        // Serialize
        byte[] serializedData = MessagePackSerializer.Serialize(animal);

        // Deserialize
        IAnimal deserializedAnimal = MessagePackSerializer.Deserialize<IAnimal>(serializedData);

        Console.WriteLine(deserializedAnimal.Name);
    }
}
```

### Explanation
1. The interface `IAnimal` is marked with `[Union]` attributes for each derived type (`Cat` and `Dog`). The numbers (`0`, `1`) are unique type identifiers used during serialization.
2. The `[MessagePackObject]` attribute is used on the base interface and derived classes to enable MessagePack serialization.
3. Serialization will encode the type information (union) along with the data, allowing deserialization to reconstruct the correct derived type.

This feature facilitates polymorphic serialization by encoding both the type identifier and the object data, making it suitable for scenarios like object-oriented network communication or storing polymorphic data structures.
## **4 - ObjectErrorAction**
![[Pasted image 20241016100745.png|center]]