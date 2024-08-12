The `Encoding` class in C# is part of the `System.Text` namespace and provides methods for encoding and decoding characters to and from bytes. Different character encodings (like UTF-8, UTF-16, ASCII, etc.) are supported, making it versatile for various text processing tasks.

### Commonly Used Properties and Methods

- **Properties**:
  - `Encoding.UTF8`: Gets an encoding for the UTF-8 format.
  - `Encoding.UTF32`: Gets an encoding for the UTF-32 format.
  - `Encoding.Unicode`: Gets an encoding for the UTF-16 format.
  - `Encoding.ASCII`: Gets an encoding for the ASCII format.

- **Methods**:
  - `GetBytes(string)`: Encodes a set of characters from a string into a byte array.
  - `GetString(byte[])`: Decodes a byte array into a string.
  - `GetBytes(char[], int, int)`: Encodes a set of characters from a character array into a byte array.
  - `GetChars(byte[], int, int)`: Decodes a byte array into a character array.
  - `GetByteCount(string)`: Calculates the number of bytes produced by encoding a set of characters from a string.
  - `GetCharCount(byte[])`: Calculates the number of characters produced by decoding a set of bytes.

### Examples

#### Encoding a String to Bytes

```csharp
using System;
using System.Text;

class Program
{
    static void Main()
    {
        string text = "Hello, Encoding!";
        
        // Get the UTF-8 encoding
        Encoding utf8 = Encoding.UTF8;

        // Convert the string to a byte array
        byte[] encodedBytes = utf8.GetBytes(text);

        // Print the encoded bytes
        Console.WriteLine("Encoded bytes: " + BitConverter.ToString(encodedBytes));
    }
}
```

#### Decoding Bytes to a String

```csharp
using System;
using System.Text;

class Program
{
    static void Main()
    {
        byte[] encodedBytes = { 72, 101, 108, 108, 111, 44, 32, 69, 110, 99, 111, 100, 105, 110, 103, 33 };
        
        // Get the UTF-8 encoding
        Encoding utf8 = Encoding.UTF8;

        // Convert the byte array to a string
        string decodedText = utf8.GetString(encodedBytes);

        // Print the decoded string
        Console.WriteLine("Decoded string: " + decodedText);
    }
}
```

#### Encoding and Decoding with Different Encodings

```csharp
using System;
using System.Text;

class Program
{
    static void Main()
    {
        string text = "Hello, Encoding!";
        
        // Get the ASCII encoding
        Encoding ascii = Encoding.ASCII;

        // Convert the string to a byte array
        byte[] asciiBytes = ascii.GetBytes(text);

        // Convert the byte array back to a string
        string decodedAsciiText = ascii.GetString(asciiBytes);

        // Print the ASCII encoded and decoded text
        Console.WriteLine("ASCII encoded and decoded: " + decodedAsciiText);
        
        // Get the Unicode encoding
        Encoding unicode = Encoding.Unicode;

        // Convert the string to a byte array
        byte[] unicodeBytes = unicode.GetBytes(text);

        // Convert the byte array back to a string
        string decodedUnicodeText = unicode.GetString(unicodeBytes);

        // Print the Unicode encoded and decoded text
        Console.WriteLine("Unicode encoded and decoded: " + decodedUnicodeText);
    }
}
```

### Summary

The `Encoding` class in C# is essential for converting between strings and byte arrays, especially when dealing with different character encodings. It's widely used in file I/O, network communication, and text processing tasks to ensure correct handling of character data.