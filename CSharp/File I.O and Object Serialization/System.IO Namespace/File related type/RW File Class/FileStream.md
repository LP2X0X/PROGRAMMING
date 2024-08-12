`FileStream` is a class in C# that provides a stream for file operations. It allows reading and writing of bytes to a file, and can handle both text and binary data. `FileStream` is part of the `System.IO` namespace and provides more control over file operations compared to higher-level classes like `StreamReader` and `StreamWriter`.

### Commonly Used Constructors

- `FileStream(string path, FileMode mode)`
- `FileStream(string path, FileMode mode, FileAccess access)`
- `FileStream(string path, FileMode mode, FileAccess access, FileShare share)`

### Commonly Used Properties and Methods

- **Properties**:
  - `CanRead`: Gets a value indicating whether the stream supports reading.
  - `CanWrite`: Gets a value indicating whether the stream supports writing.
  - `CanSeek`: Gets a value indicating whether the stream supports seeking.
  - `Length`: Gets the length in bytes of the stream.
  - `Position`: Gets or sets the current position within the stream.

- **Methods**:
  - `Read(byte[], int, int)`: Reads a block of bytes from the stream and writes the data to a buffer.
  - `Write(byte[], int, int)`: Writes a block of bytes to the stream.
  - `Seek(long, SeekOrigin)`: Sets the position within the current stream.
  - `Flush()`: Clears all buffers for the stream and causes any buffered data to be written to the file.
  - `Close()`: Closes the current stream and releases any resources associated with it.

### Examples

#### Reading from a File

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        string path = "example.txt";

        using (FileStream fs = new FileStream(path, FileMode.Open, FileAccess.Read))
        {
            byte[] buffer = new byte[1024];
            int bytesRead = fs.Read(buffer, 0, buffer.Length);

            string content = System.Text.Encoding.UTF8.GetString(buffer, 0, bytesRead);
            Console.WriteLine(content);
        }
    }
}
```

#### Writing to a File

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        string path = "example.txt";
        string content = "Hello, FileStream!";

        using (FileStream fs = new FileStream(path, FileMode.Create, FileAccess.Write))
        {
            byte[] buffer = System.Text.Encoding.UTF8.GetBytes(content);
            fs.Write(buffer, 0, buffer.Length);
        }
    }
}
```

#### Appending to a File

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        string path = "example.txt";
        string content = "\nAppending more content.";

        using (FileStream fs = new FileStream(path, FileMode.Append, FileAccess.Write))
        {
            byte[] buffer = System.Text.Encoding.UTF8.GetBytes(content);
            fs.Write(buffer, 0, buffer.Length);
        }
    }
}
```

#### Seeking Within a File

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        string path = "example.txt";
        string content = "1234567890";

        // Write content to file
        using (FileStream fs = new FileStream(path, FileMode.Create, FileAccess.Write))
        {
            byte[] buffer = System.Text.Encoding.UTF8.GetBytes(content);
            fs.Write(buffer, 0, buffer.Length);
        }

        // Read from a specific position
        using (FileStream fs = new FileStream(path, FileMode.Open, FileAccess.Read))
        {
            fs.Seek(4, SeekOrigin.Begin); // Seek to the 5th byte
            byte[] buffer = new byte[3];
            int bytesRead = fs.Read(buffer, 0, buffer.Length);

            string partialContent = System.Text.Encoding.UTF8.GetString(buffer, 0, bytesRead);
            Console.WriteLine("Partial content: " + partialContent); // Outputs "567"
        }
    }
}
```

### Summary

`FileStream` is a versatile class for handling file I/O operations at a low level. It provides fine-grained control over how data is read from and written to files, making it suitable for both binary and text data. The use of `FileStream` allows for efficient file manipulation and is a fundamental part of file I/O in C#.