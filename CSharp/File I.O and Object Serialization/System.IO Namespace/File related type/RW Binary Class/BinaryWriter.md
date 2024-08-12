`BinaryReader` and `BinaryWriter` are classes in C# that facilitate reading and writing primitive data types in binary format from and to a stream. They are part of the `System.IO` namespace and are useful for handling binary files or any stream of bytes.

### BinaryWriter

`BinaryWriter` is used to write binary data to a stream. It supports writing primitive data types such as integers, floats, doubles, booleans, and strings.

#### Commonly Used Constructors

- `BinaryWriter(Stream output)`
- `BinaryWriter(Stream output, Encoding encoding)`

#### Commonly Used Methods

- `Write(bool value)`: Writes a boolean value to the current stream and advances the position within the stream by one byte.
- `Write(byte value)`: Writes an unsigned byte to the current stream and advances the position within the stream by one byte.
- `Write(int value)`: Writes a 4-byte signed integer to the current stream and advances the position within the stream by four bytes.
- `Write(double value)`: Writes an 8-byte floating point value to the current stream and advances the position within the stream by eight bytes.
- `Write(string value)`: Writes a length-prefixed string to this stream in the current encoding of the `BinaryWriter`.

#### Example of `BinaryWriter`

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        string filePath = "example.bin";

        // Write binary data to a file
        using (BinaryWriter writer = new BinaryWriter(File.Open(filePath, FileMode.Create)))
        {
            writer.Write(1.2345);      // Write a double
            writer.Write(42);          // Write an int
            writer.Write("Hello");     // Write a string
        }

        Console.WriteLine("Binary data written to file.");
    }
}
```

### Summary

- **BinaryReader**: Used for reading binary data from a stream. It can read primitive data types and strings.
- **BinaryWriter**: Used for writing binary data to a stream. It can write primitive data types and strings.
- **Use Cases**: Ideal for reading and writing data in binary formats, such as files with custom binary structures, network protocols, or any situation where performance and storage efficiency are critical.
- **Efficiency**: Both classes are efficient for handling binary data, allowing for fast reading and writing operations.