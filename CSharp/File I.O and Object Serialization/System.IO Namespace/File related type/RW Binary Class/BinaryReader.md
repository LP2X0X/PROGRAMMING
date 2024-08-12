`BinaryReader` and `BinaryWriter` are classes in C# that facilitate reading and writing primitive data types in binary format from and to a stream. They are part of the `System.IO` namespace and are useful for handling binary files or any stream of bytes.

### BinaryReader

`BinaryReader` is used to read binary data from a stream. It supports reading primitive data types such as integers, floats, doubles, booleans, and strings.

#### Commonly Used Constructors

- `BinaryReader(Stream input)`
- `BinaryReader(Stream input, Encoding encoding)`

#### Commonly Used Methods

- `ReadBoolean()`: Reads a boolean value from the current stream and advances the position of the stream by one byte.
- `ReadByte()`: Reads the next byte from the current stream and advances the position of the stream by one byte.
- `ReadInt32()`: Reads a 4-byte signed integer from the current stream and advances the position of the stream by four bytes.
- `ReadDouble()`: Reads an 8-byte floating point value from the current stream and advances the position of the stream by eight bytes.
- `ReadString()`: Reads a string from the current stream. The string is prefixed with the length, encoded as an integer.

#### Example of `BinaryReader`

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        string filePath = "example.bin";

        // Create a file with some binary data
        using (BinaryWriter writer = new BinaryWriter(File.Open(filePath, FileMode.Create)))
        {
            writer.Write(1.2345);      // Write a double
            writer.Write(42);          // Write an int
            writer.Write("Hello");     // Write a string
        }

        // Read the binary data from the file
        using (BinaryReader reader = new BinaryReader(File.Open(filePath, FileMode.Open)))
        {
            double doubleValue = reader.ReadDouble();
            int intValue = reader.ReadInt32();
            string stringValue = reader.ReadString();

            Console.WriteLine($"Read Double: {doubleValue}");
            Console.WriteLine($"Read Int: {intValue}");
            Console.WriteLine($"Read String: {stringValue}");
        }
    }
}
```

### Summary

- **BinaryReader**: Used for reading binary data from a stream. It can read primitive data types and strings.
- **BinaryWriter**: Used for writing binary data to a stream. It can write primitive data types and strings.
- **Use Cases**: Ideal for reading and writing data in binary formats, such as files with custom binary structures, network protocols, or any situation where performance and storage efficiency are critical.
- **Efficiency**: Both classes are efficient for handling binary data, allowing for fast reading and writing operations.