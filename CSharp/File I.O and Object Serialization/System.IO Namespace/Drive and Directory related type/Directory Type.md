In .NET, the `Directory` class in the `System.IO` namespace provides static methods for creating, moving, and enumerating through directories and subdirectories. Unlike the `DirectoryInfo` class, which is used to create instances representing directories, the `Directory` class provides static methods and does not require you to create an instance.

Here are some common methods of the `Directory` class:

### Common Methods:

#### Directory Creation and Deletion
- `CreateDirectory(string path)`: Creates all directories and subdirectories in the specified path unless they already exist.
- `Delete(string path)`: Deletes an empty directory from a specified path.
- `Delete(string path, bool recursive)`: Deletes the specified directory and, if indicated, any subdirectories and files in the directory.

#### Directory Movement and Renaming
- `Move(string sourceDirName, string destDirName)`: Moves a file or a directory and its contents to a new location.

#### Directory Information and Enumeration
- `Exists(string path)`: Determines whether the given path refers to an existing directory.
- `GetFiles(string path)`: Returns the names of files in the specified directory.
- `GetFiles(string path, string searchPattern)`: Returns the names of files that match the specified search pattern in the specified directory.
- `GetDirectories(string path)`: Returns the names of subdirectories in the specified directory.
- `GetDirectories(string path, string searchPattern)`: Returns the names of subdirectories that match the specified search pattern in the specified directory.
- `GetFileSystemEntries(string path)`: Returns the names of all files and subdirectories in the specified directory.
- `GetFileSystemEntries(string path, string searchPattern)`: Returns the names of files and subdirectories that match the specified search pattern in the specified directory.

#### Example:

Here is an example demonstrating some of the `Directory` class methods:

```csharp
using System;
using System.IO;

class Program
{
    static void Main()
    {
        string path = @"C:\ExampleDir";

        // Create a new directory if it does not exist
        if (!Directory.Exists(path))
        {
            Directory.CreateDirectory(path);
            Console.WriteLine("Directory created.");
        }

        // List all files in the directory
        string[] files = Directory.GetFiles(path);
        Console.WriteLine("Files in directory:");
        foreach (string file in files)
        {
            Console.WriteLine(file);
        }

        // List all subdirectories in the directory
        string[] directories = Directory.GetDirectories(path);
        Console.WriteLine("Subdirectories in directory:");
        foreach (string directory in directories)
        {
            Console.WriteLine(directory);
        }

        // Move the directory to a new location
        string newPath = @"C:\NewExampleDir";
        if (!Directory.Exists(newPath))
        {
            Directory.Move(path, newPath);
            Console.WriteLine("Directory moved.");
        }

        // Delete the directory
        if (Directory.Exists(newPath))
        {
            Directory.Delete(newPath, true); // 'true' for recursive deletion
            Console.WriteLine("Directory deleted.");
        }
    }
}
```

In this example:
- A new directory is created if it doesn't already exist.
- All files and subdirectories within the directory are listed.
- The directory is moved to a new location.
- The directory is then deleted.

The `Directory` class is ideal for quick and straightforward directory operations where instance methods are unnecessary.