---
tags:
 - csharp
 - io
 - directories
---

## Directory and DirectoryInfo

These two types handle all directory (folder) operations in `System.IO`. They follow the [[System.IO Namespace Overview|static vs instance pattern]] -- `Directory` is a static utility class for quick one-shot operations, while `DirectoryInfo` is an instance class that wraps a directory path into a reusable object with rich metadata.

---

## Directory (Static Class)

`Directory` provides static methods for creating, deleting, moving, and enumerating directories. Every method takes a path string and performs the operation immediately.

### Creating Directories

```csharp
// Creates the directory and ALL intermediate directories in the path.
// If the directory already exists, this is a no-op -- no exception thrown.
Directory.CreateDirectory(@"C:\Projects\MyApp\Logs\Archive");
// This creates Projects, MyApp, Logs, AND Archive if any are missing.
```

```ad-note
title: CreateDirectory is Idempotent
Unlike `File.Create()` which overwrites, `Directory.CreateDirectory()` does nothing if the directory already exists. This makes it safe to call without checking `Exists()` first.
```

### Checking Existence

```csharp
bool exists = Directory.Exists(@"C:\Projects\MyApp");
// Returns false for files -- only true for directories
// Returns false if the path is null or empty (no exception)
```

### Listing Contents

This is where `Directory` really shines. There are two families of methods, and understanding the difference is important.

#### Get Methods (Eager -- Returns Arrays)

```csharp
// Returns ALL matching file paths as string[] -- entire array in memory at once
string[] txtFiles = Directory.GetFiles(@"C:\Data", "*.txt");

// Returns ALL subdirectory paths as string[]
string[] subDirs = Directory.GetDirectories(@"C:\Data");

// Returns both files AND directories as string[]
string[] everything = Directory.GetFileSystemEntries(@"C:\Data");
```

#### Enumerate Methods (Lazy -- Returns IEnumerable)

```csharp
// Returns an IEnumerable<string> -- items yielded one at a time
// Much better for large directories because you don't load everything into memory
IEnumerable<string> txtFiles = Directory.EnumerateFiles(@"C:\Data", "*.txt");

IEnumerable<string> subDirs = Directory.EnumerateDirectories(@"C:\Data");

IEnumerable<string> everything = Directory.EnumerateFileSystemEntries(@"C:\Data");
```

| Method Family | Return Type | Memory | Best For |
|---|---|---|---|
| `GetFiles()` / `GetDirectories()` | `string[]` | Entire array allocated up front | Small directories, when you need `.Length` or random access |
| `EnumerateFiles()` / `EnumerateDirectories()` | `IEnumerable<string>` | Items yielded lazily | Large directories, LINQ pipelines, early exit with `.First()` |

```ad-important
title: Prefer Enumerate for Large Directories
If a directory has 100,000 files, `GetFiles()` allocates a 100,000-element array before you see a single result. `EnumerateFiles()` starts yielding results immediately and uses negligible memory. Always prefer `Enumerate*` unless you specifically need an array.
```

#### Search Options

Both families accept an optional `SearchOption` parameter:

```csharp
// TopDirectoryOnly (default) -- only immediate children
string[] topOnly = Directory.GetFiles(@"C:\Data", "*.*", SearchOption.TopDirectoryOnly);

// AllDirectories -- recursive search through entire subtree
string[] recursive = Directory.GetFiles(@"C:\Data", "*.log", SearchOption.AllDirectories);
```

In .NET 5+, there is also an `EnumerationOptions` overload for fine-grained control:

```csharp
var options = new EnumerationOptions
{
    RecurseSubdirectories = true,
    IgnoreInaccessible = true,          // skip dirs you can't read instead of throwing
    MatchCasing = MatchCasing.CaseInsensitive,
    AttributesToSkip = FileAttributes.Hidden | FileAttributes.System
};

foreach (string file in Directory.EnumerateFiles(@"C:\Data", "*.txt", options))
{
    Console.WriteLine(file);
}
```

```ad-note
title: EnumerationOptions.IgnoreInaccessible
This is extremely useful. Without it, enumerating a directory tree that contains a folder you don't have permission to read throws an `UnauthorizedAccessException` and stops the entire enumeration. With `IgnoreInaccessible = true`, those folders are silently skipped.
```

#### Pattern Matching

The search pattern supports basic wildcards:

| Pattern | Matches |
|---|---|
| `*.*` | All files with an extension |
| `*.txt` | All `.txt` files |
| `report_*` | Files starting with `report_` |
| `???.log` | Files with exactly 3-character names and `.log` extension |
| `*` | Everything (files and/or directories depending on method) |

```ad-warning
title: Pattern Matching Quirk with 3-Character Extensions
The `*.txt` pattern also matches files with extensions that START with `.txt`, like `.txtx` or `.txt_backup` on some older .NET Framework versions. In .NET Core/.NET 5+, this is fixed to match exactly. If targeting older frameworks, be aware of this edge case.
```

### Moving and Renaming

```csharp
// Move a directory to a new location (this is also how you rename)
Directory.Move(@"C:\OldName", @"C:\NewName");

// Moving to a different drive copies + deletes (slower)
Directory.Move(@"C:\Data", @"D:\Data");
```

```ad-warning
title: Directory.Move Constraints
- The destination must NOT already exist -- throws `IOException` if it does.
- You cannot move a directory to a different volume on some platforms (works on Windows, may fail on Linux).
- Moving a directory that is the current working directory of any process throws `IOException`.
```

### Deleting

```csharp
// Delete an EMPTY directory
Directory.Delete(@"C:\Temp\OldDir");

// Delete a directory and ALL its contents recursively
Directory.Delete(@"C:\Temp\OldDir", recursive: true);
```

```ad-warning
title: Recursive Delete is Dangerous
`Directory.Delete(path, true)` permanently deletes ALL files and subdirectories with no confirmation and no recycle bin. Double-check your path. Consider logging or confirming before calling this in production code. There is no undo.
```

### Other Useful Methods

```csharp
// Get/set current working directory
string cwd = Directory.GetCurrentDirectory();
Directory.SetCurrentDirectory(@"C:\Projects");

// Get/set timestamps
DateTime created = Directory.GetCreationTime(@"C:\Data");
Directory.SetLastWriteTime(@"C:\Data", DateTime.Now);

// Get the logical drives on the system
string[] drives = Directory.GetLogicalDrives(); // ["C:\", "D:\", ...]
```

---

## DirectoryInfo (Instance Class)

`DirectoryInfo` wraps a directory path into an object. You create one instance and then access properties and methods on it repeatedly.

### Construction

```csharp
// Constructor takes a path string -- does NOT verify the directory exists
DirectoryInfo dir = new DirectoryInfo(@"C:\Projects\MyApp");
```

### Key Properties

| Property | Type | Description |
|---|---|---|
| `Name` | `string` | Directory name only (e.g., `"MyApp"`) |
| `FullName` | `string` | Full absolute path (e.g., `@"C:\Projects\MyApp"`) |
| `Parent` | `DirectoryInfo?` | Parent directory as a `DirectoryInfo` object, or `null` for root |
| `Root` | `DirectoryInfo` | Root of the path (e.g., `C:\`) |
| `Exists` | `bool` | Whether the directory currently exists on disk |
| `CreationTime` | `DateTime` | When the directory was created |
| `LastWriteTime` | `DateTime` | When the directory was last modified |
| `LastAccessTime` | `DateTime` | When the directory was last accessed |
| `Attributes` | `FileAttributes` | Directory attributes (Hidden, ReadOnly, etc.) |

### Key Methods

| Method | Return Type | Description |
|---|---|---|
| `Create()` | `void` | Creates the directory (including intermediates) |
| `Delete()` | `void` | Deletes if empty |
| `Delete(true)` | `void` | Deletes recursively |
| `MoveTo(destPath)` | `void` | Moves/renames the directory |
| `GetFiles()` | `FileInfo[]` | Returns child files as `FileInfo` objects |
| `GetFiles(pattern)` | `FileInfo[]` | Returns matching child files |
| `GetFiles(pattern, searchOption)` | `FileInfo[]` | Optionally recursive |
| `GetDirectories()` | `DirectoryInfo[]` | Returns subdirectories as `DirectoryInfo` objects |
| `EnumerateFiles()` | `IEnumerable<FileInfo>` | Lazy enumeration of child files |
| `EnumerateDirectories()` | `IEnumerable<DirectoryInfo>` | Lazy enumeration of subdirectories |
| `Refresh()` | `void` | Re-reads cached metadata from disk |

```ad-note
title: DirectoryInfo Returns Typed Objects
The key difference from the static `Directory` class: `DirectoryInfo.GetFiles()` returns `FileInfo[]` (objects with properties like `Length`, `Extension`, `CreationTime`), while `Directory.GetFiles()` returns `string[]` (just paths). If you need metadata about each file, `DirectoryInfo` saves you from constructing `FileInfo` objects yourself.
```

### Example: DirectoryInfo in Action

```csharp
DirectoryInfo dir = new DirectoryInfo(@"C:\Projects\MyApp");

if (!dir.Exists)
{
    dir.Create();
    Console.WriteLine($"Created: {dir.FullName}");
}

Console.WriteLine($"Name:          {dir.Name}");
Console.WriteLine($"Full Path:     {dir.FullName}");
Console.WriteLine($"Parent:        {dir.Parent?.FullName}");
Console.WriteLine($"Created:       {dir.CreationTime}");
Console.WriteLine($"Last Modified: {dir.LastWriteTime}");

// List all .cs files recursively, sorted by size descending
FileInfo[] csFiles = dir.GetFiles("*.cs", SearchOption.AllDirectories);
foreach (FileInfo file in csFiles.OrderByDescending(f => f.Length))
{
    Console.WriteLine($"  {file.FullName} ({file.Length:N0} bytes)");
}

// List immediate subdirectories
foreach (DirectoryInfo subDir in dir.GetDirectories())
{
    Console.WriteLine($"  [DIR] {subDir.Name}");
}
```

---

## Getting Special System Folders

The `Environment` class (not in `System.IO` but essential for directory work) provides access to well-known system folders:

```csharp
// User-specific folders
string desktop   = Environment.GetFolderPath(Environment.SpecialFolder.Desktop);
string documents = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
string appData   = Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData);    // Roaming
string localData = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData); // Local

// System folders
string programFiles = Environment.GetFolderPath(Environment.SpecialFolder.ProgramFiles);
string system32     = Environment.GetFolderPath(Environment.SpecialFolder.System);
string temp         = Path.GetTempPath(); // typically C:\Users\<user>\AppData\Local\Temp
```

```ad-info
title: Common SpecialFolder Values
- `ApplicationData` -- roaming app data, synced across machines in a domain (`%APPDATA%`)
- `LocalApplicationData` -- local app data, machine-specific (`%LOCALAPPDATA%`)
- `CommonApplicationData` -- shared across all users (`%PROGRAMDATA%`)
- `Desktop`, `MyDocuments`, `MyMusic`, `MyPictures` -- user profile folders
- `Startup` -- current user's startup folder
```

---

## Complete Working Example

This example creates a directory structure, populates it with files, lists everything, and cleans up:

```csharp
using System;
using System.IO;

string basePath = Path.Combine(Path.GetTempPath(), "DirectoryDemo");

try
{
    // 1. Create a directory tree
    DirectoryInfo baseDir = Directory.CreateDirectory(basePath);
    Directory.CreateDirectory(Path.Combine(basePath, "Logs"));
    Directory.CreateDirectory(Path.Combine(basePath, "Data", "Raw"));   // nested creation
    Directory.CreateDirectory(Path.Combine(basePath, "Data", "Processed"));

    Console.WriteLine($"Created: {baseDir.FullName}");

    // 2. Create some dummy files
    File.WriteAllText(Path.Combine(basePath, "readme.txt"), "Hello!");
    File.WriteAllText(Path.Combine(basePath, "Logs", "app.log"), "Log entry 1");
    File.WriteAllText(Path.Combine(basePath, "Data", "Raw", "input.csv"), "a,b,c");

    // 3. Enumerate everything recursively
    Console.WriteLine("\nAll files:");
    foreach (string file in Directory.EnumerateFiles(basePath, "*", SearchOption.AllDirectories))
    {
        Console.WriteLine($"  {Path.GetRelativePath(basePath, file)}");
    }

    Console.WriteLine("\nAll directories:");
    foreach (string dir in Directory.EnumerateDirectories(basePath, "*", SearchOption.AllDirectories))
    {
        Console.WriteLine($"  {Path.GetRelativePath(basePath, dir)}");
    }

    // 4. Use DirectoryInfo for metadata
    DirectoryInfo dataDir = new DirectoryInfo(Path.Combine(basePath, "Data"));
    Console.WriteLine($"\nData dir has {dataDir.GetFiles("*", SearchOption.AllDirectories).Length} file(s)");
    Console.WriteLine($"Data dir has {dataDir.GetDirectories().Length} subdirectories");
}
finally
{
    // 5. Clean up -- recursive delete
    if (Directory.Exists(basePath))
    {
        Directory.Delete(basePath, recursive: true);
        Console.WriteLine($"\nCleaned up: {basePath}");
    }
}
```

---

## Summary

- **`Directory`** (static) is for quick, one-shot directory operations -- create, check existence, list files, delete.
- **`DirectoryInfo`** (instance) wraps a path into a reusable object with metadata properties and methods that return strongly-typed `FileInfo[]`/`DirectoryInfo[]`.
- Prefer **`EnumerateFiles()`** over `GetFiles()` for large directories -- lazy enumeration avoids loading everything into memory.
- Use **`EnumerationOptions`** with `IgnoreInaccessible = true` for robust recursive enumeration that won't crash on permission-denied folders.
- `Directory.CreateDirectory()` is safe to call even if the directory exists -- it's a no-op.
- `Directory.Delete(path, true)` is permanent and recursive with **no undo** -- use with extreme caution.
