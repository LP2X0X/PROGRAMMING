In C#, `FileStream` and `StreamReader` are classes provided by the .NET framework for reading from and writing to files. Here’s a detailed look at both:

### FileStream

`FileStream` provides a way to read from and write to any location within a file.

#### Example of `FileStream` for Reading

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

#### Example of `FileStream` for Writing

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

### StreamReader

`StreamReader` is a convenience class for reading characters from a byte stream in a particular encoding.

#### Example of `StreamReader`

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        string path = "example.txt";

        using (StreamReader sr = new StreamReader(path))
        {
            string content = sr.ReadToEnd();
            Console.WriteLine(content);
        }
    }
}
```

### Differences and Use Cases

1. **Purpose**:
   - `FileStream` is more general-purpose and can be used for reading and writing bytes. It's suitable for both binary and text data.
   - `StreamReader` is specifically designed for reading text and provides higher-level methods for reading strings, lines, and characters.

2. **Encoding**:
   - `FileStream` reads and writes raw bytes. Handling character encoding is manual and usually requires additional steps.
   - `StreamReader` handles character encoding internally, making it easier to read text files with different encodings.

3. **Performance**:
   - `FileStream` may be more efficient for large binary files since it works with byte arrays.
   - `StreamReader` is more convenient for reading text data and handling line breaks and character encoding automatically.

### Example: Using Both Together

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        string path = "example.txt";

        // Writing with FileStream
        using (FileStream fs = new FileStream(path, FileMode.Create, FileAccess.Write))
        {
            string content = "Hello, FileStream and StreamReader!";
            byte[] buffer = System.Text.Encoding.UTF8.GetBytes(content);
            fs.Write(buffer, 0, buffer.Length);
        }

        // Reading with StreamReader
        using (StreamReader sr = new StreamReader(path))
        {
            string content = sr.ReadToEnd();
            Console.WriteLine(content);
        }
    }
}
```

In this example, `FileStream` is used for writing bytes to the file, and `StreamReader` is used for reading text from the file. This combination leverages the strengths of both classes.