In .NET, particularly in C#, the `System.IO` namespace provides classes for working with the file system. Two important classes for handling directories and files are `DirectoryInfo` and `FileInfo`. 

### `DirectoryInfo` Class

The `DirectoryInfo` class provides instance methods for creating, moving, and enumerating through directories and subdirectories. Here are some common properties and methods:

#### Common Properties:
- `FullName`: Gets the full path of the directory.
- `Name`: Gets the name of the directory.
- `Parent`: Gets the parent directory of a specified subdirectory.
- `Exists`: Gets a value indicating whether the directory exists.

#### Common Methods:
- `Create()`: Creates the directory.
- `Delete()`: Deletes the directory.
- `GetFiles()`: Returns a file list from the current directory.
- `GetDirectories()`: Returns the subdirectories of the current directory.
- `MoveTo()`: Moves a directory and its contents to a new path.
- `EnumerateFiles()`: Returns an enumerable collection of file information in the directory.
- `EnumerateDirectories()`: Returns an enumerable collection of directory information in the directory.

#### Example:
```csharp
DirectoryInfo dir = new DirectoryInfo(@"C:\ExampleDir");
if (!dir.Exists)
{
    dir.Create();
}

FileInfo[] files = dir.GetFiles();
foreach (FileInfo file in files)
{
    Console.WriteLine(file.Name);
}
```

### `FileInfo` Class

The `FileInfo` class provides instance methods for the creation, copying, deletion, moving, and opening of files, and aids in the creation of `FileStream` objects.

#### Common Properties:
- `FullName`: Gets the full path of the file.
- `Name`: Gets the name of the file.
- `Directory`: Gets an instance of the parent directory.
- `Exists`: Gets a value indicating whether the file exists.
- `Length`: Gets the size of the file.

#### Common Methods:
- `Create()`: Creates a file.
- `Delete()`: Deletes a file.
- `CopyTo()`: Copies an existing file to a new file.
- `MoveTo()`: Moves a specified file to a new location.
- `Open()`: Opens a file in the specified mode.

#### Example:
```csharp
FileInfo file = new FileInfo(@"C:\ExampleDir\example.txt");
if (!file.Exists)
{
    using (StreamWriter sw = file.CreateText())
    {
        sw.WriteLine("Hello, World!");
    }
}

Console.WriteLine("File Size: " + file.Length);
```

### Using `DirectoryInfo` and `FileInfo` Together

You can use these classes together to perform complex file system operations. For example, you can list all files in a directory and process each file:

```csharp
DirectoryInfo dir = new DirectoryInfo(@"C:\ExampleDir");
if (dir.Exists)
{
    foreach (FileInfo file in dir.GetFiles())
    {
        Console.WriteLine($"Processing file: {file.Name}");
        // Perform file operations here
    }
}
```

These classes offer a higher level of abstraction compared to static methods in the `File` and `Directory` classes, providing more object-oriented ways to handle file system operations.