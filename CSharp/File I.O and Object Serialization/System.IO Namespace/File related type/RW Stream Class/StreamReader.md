`StreamWriter` and `StreamReader` are classes in C# provided by the .NET framework to handle text-based input and output operations. They are part of the `System.IO` namespace and are specifically designed for working with text files, handling character encoding automatically.

```ad-note
Derived from abstract type [[TextReader]]
```

### StreamReader

`StreamReader` is used for reading characters from a stream in a particular encoding.

#### Commonly Used Constructors

- `StreamReader(string path)`
- `StreamReader(string path, Encoding encoding)`

#### Commonly Used Methods

- `Read()`: Reads the next character from the input stream and advances the character position by one character.
- `ReadLine()`: Reads a line of characters from the current stream and returns the data as a string.
- `ReadToEnd()`: Reads the stream from the current position to the end of the stream.
- `Close()`: Closes the `StreamReader` object and the underlying stream.

#### Example of `StreamReader`

```csharp
using System;
using System.IO;
using System.Text;

class Program
{
    static void Main()
    {
        string path = "example.txt";

        using (StreamReader sr = new StreamReader(path, Encoding.UTF8))
        {
            string content = sr.ReadToEnd();
            Console.WriteLine("File content:");
            Console.WriteLine(content);
        }
    }
}
```

### Combined Example

Here's an example of writing to a file using `StreamWriter` and then reading from the same file using `StreamReader`.

```csharp
using System;
using System.IO;
using System.Text;

class Program
{
    static void Main()
    {
        string path = "example.txt";
        string contentToWrite = "Hello, StreamWriter and StreamReader!";

        // Write to the file
        using (StreamWriter sw = new StreamWriter(path, false, Encoding.UTF8))
        {
            sw.WriteLine(contentToWrite);
        }

        Console.WriteLine("Content written to file.");

        // Read from the file
        using (StreamReader sr = new StreamReader(path, Encoding.UTF8))
        {
            string contentRead = sr.ReadToEnd();
            Console.WriteLine("File content:");
            Console.WriteLine(contentRead);
        }
    }
}
```

### Summary

- `StreamWriter` and `StreamReader` are high-level classes for handling text files in C#.
- `StreamWriter` is used for writing text data to a file, and `StreamReader` is used for reading text data from a file.
- They automatically handle character encoding, making it easy to work with text files in different encodings.
- They simplify many common file I/O tasks, such as writing lines of text or reading the entire contents of a file into a string.