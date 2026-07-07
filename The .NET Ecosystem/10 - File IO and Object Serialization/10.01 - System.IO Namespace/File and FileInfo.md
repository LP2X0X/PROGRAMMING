---
tags:
 - csharp
 - io
 - files
---

## File and FileInfo

These are the two primary types for working with files in `System.IO`. Like the [[Directory and DirectoryInfo|directory counterparts]], they follow the [[System.IO Namespace Overview|static vs instance pattern]] -- `File` is a static utility class for quick operations, `FileInfo` is an instance class for reusable, metadata-rich file objects.

---

## File (Static Class)

### Quick Read Methods

These are the **most commonly used** methods in all of `System.IO`. Each one opens the file, reads the content, and closes the handle in a single call.

```csharp
// Read entire file as a single string
string content = File.ReadAllText(@"C:\data\config.json");

// Read entire file as a single string with specific encoding
string content = File.ReadAllText(@"C:\data\config.json", Encoding.UTF8);

// Read file into an array of lines (splits on \r\n, \r, or \n)
string[] lines = File.ReadAllLines(@"C:\data\log.csv");

// Read file into a byte array (for binary data)
byte[] bytes = File.ReadAllBytes(@"C:\data\image.png");

// .NET 6+: Async versions
string content = await File.ReadAllTextAsync("config.json");
string[] lines = await File.ReadAllLinesAsync("data.csv");
byte[] bytes = await File.ReadAllBytesAsync("image.png");
```

```ad-warning
title: ReadAllText/ReadAllBytes Load the ENTIRE File Into Memory
For a 2 GB log file, `File.ReadAllText()` tries to allocate a 2 GB string. For large files, use streaming with `StreamReader` or `File.ReadLines()` instead.
```

**`File.ReadLines()` -- The Lazy Alternative to ReadAllLines:**

```csharp
// ReadAllLines: reads entire file, returns string[] -- all in memory at once
string[] allLines = File.ReadAllLines("huge.csv");

// ReadLines: returns IEnumerable<string> -- reads lazily, one line at a time
// Much better for large files or when you only need a few lines
foreach (string line in File.ReadLines("huge.csv"))
{
    if (line.StartsWith("ERROR"))
    {
        Console.WriteLine(line);
        break; // stops reading here -- doesn't process the rest of the file
    }
}
```

### Quick Write Methods

```csharp
// Write a string to a file -- creates or OVERWRITES the file
File.WriteAllText("output.txt", "Hello, World!");

// Write an array of lines -- each element becomes one line
File.WriteAllLines("data.csv", new[] { "Name,Age", "Alice,30", "Bob,25" });

// Write a byte array -- for binary data
File.WriteAllBytes("output.bin", new byte[] { 0x48, 0x65, 0x6C, 0x6C, 0x6F });

// APPEND text to end of file (creates if doesn't exist)
File.AppendAllText("log.txt", $"[{DateTime.Now:O}] Application started\n");

// APPEND lines to end of file
File.AppendAllLines("log.txt", new[] { "Line 1", "Line 2" });

// .NET 6+: Async versions
await File.WriteAllTextAsync("output.txt", content);
await File.AppendAllTextAsync("log.txt", message);
```

```ad-note
title: WriteAllText vs AppendAllText
- `WriteAllText` **overwrites** the entire file if it exists (equivalent to `FileMode.Create`).
- `AppendAllText` **appends** to the end of the file, preserving existing content (equivalent to `FileMode.Append`).
```

### Checking Existence

```csharp
bool exists = File.Exists(@"C:\data\config.json");
// Returns false if path is null, empty, or points to a directory
// Never throws an exception -- always returns true or false
```

### Copy, Move, Delete

```csharp
// Copy a file
File.Copy("source.txt", "destination.txt");

// Copy with overwrite (without this, throws IOException if dest exists)
File.Copy("source.txt", "destination.txt", overwrite: true);

// Move a file (also serves as rename)
File.Move("old.txt", "new.txt");

// Move with overwrite (.NET 5+ only -- earlier versions throw if dest exists)
File.Move("old.txt", "new.txt", overwrite: true);

// Delete a file -- no exception if file doesn't exist
File.Delete("temp.txt");
```

```ad-warning
title: File.Delete Quirks
- `File.Delete()` does NOT throw if the file doesn't exist -- it's a silent no-op.
- It DOES throw `UnauthorizedAccessException` if the file is read-only. Set `File.SetAttributes(path, FileAttributes.Normal)` first, or use `FileInfo` and set `IsReadOnly = false`.
- It DOES throw `IOException` if the file is locked by another process.
- Deleted files do NOT go to the recycle bin -- they are permanently removed.
```

```ad-info
title: File.Move Before .NET 5
In .NET Framework and .NET Core < 5, `File.Move()` has no overwrite parameter. If the destination file exists, it throws `IOException`. You must delete the destination first:

    File.Delete("new.txt");
    File.Move("old.txt", "new.txt");
```

---

## Opening Streams from the File Class

When you need more control than the quick read/write methods provide, `File` has methods that return streams or readers/writers.

### File.Open -- Full Control

```csharp
// Full control: specify mode, access, and sharing
using FileStream fs = File.Open("data.bin", FileMode.OpenOrCreate, FileAccess.ReadWrite, FileShare.None);
```

### Convenience Open Methods

```csharp
// Open for reading only (FileMode.Open, FileAccess.Read, FileShare.Read)
using FileStream readStream = File.OpenRead("data.bin");

// Open for writing only (FileMode.OpenOrCreate, FileAccess.Write, FileShare.None)
using FileStream writeStream = File.OpenWrite("data.bin");

// Open for reading text (returns StreamReader)
using StreamReader reader = File.OpenText("data.txt");

// Create for writing text (returns StreamWriter -- creates or overwrites)
using StreamWriter writer = File.CreateText("output.txt");

// Open for appending text (returns StreamWriter)
using StreamWriter appender = File.AppendText("log.txt");

// Create a new file (returns FileStream -- creates or overwrites)
using FileStream newFile = File.Create("newfile.dat");
```

---

## FileMode Enum

`FileMode` controls **how** the operating system opens or creates the file.

| Value | Description | File Exists | File Doesn't Exist |
|---|---|---|---|
| `CreateNew` | Create a new file; fail if it exists | Throws `IOException` | Creates new file |
| `Create` | Create new or overwrite existing | Truncates to zero length | Creates new file |
| `Open` | Open existing; fail if it doesn't exist | Opens file | Throws `FileNotFoundException` |
| `OpenOrCreate` | Open if exists, create if not | Opens file | Creates new file |
| `Truncate` | Open existing and truncate to zero; fail if doesn't exist | Opens and empties | Throws `FileNotFoundException` |
| `Append` | Open for appending; create if doesn't exist | Opens, seeks to end | Creates new file |

```ad-important
title: Create vs CreateNew
- `FileMode.Create` silently overwrites an existing file. Data is gone.
- `FileMode.CreateNew` throws if the file already exists, preventing accidental overwrites.

Use `CreateNew` when you want to ensure you're not clobbering an existing file. Use `Create` when you intentionally want "create-or-overwrite" behavior.
```

---

## FileAccess Enum

`FileAccess` controls **what operations** you can perform on the stream.

| Value | Description |
|---|---|
| `Read` | Read-only access |
| `Write` | Write-only access |
| `ReadWrite` | Both read and write access |

```csharp
// Open a file for reading only -- Write() calls will throw
using var fs = new FileStream("data.txt", FileMode.Open, FileAccess.Read);
```

---

## FileShare Enum

`FileShare` controls **what other processes** can do with the file while you have it open. This is critical for multi-process scenarios.

| Value | Description |
|---|---|
| `None` | No other process can open the file (exclusive lock) |
| `Read` | Other processes can open for reading |
| `Write` | Other processes can open for writing |
| `ReadWrite` | Other processes can open for reading or writing |
| `Delete` | Other processes can delete the file |

```csharp
// Allow other processes to read while we write (common for log files)
using var fs = new FileStream("app.log", FileMode.Append, FileAccess.Write, FileShare.Read);
```

```ad-note
title: FileShare.Read is the Log File Pattern
When writing a log file, use `FileShare.Read` so that log viewers (like `tail -f` or a monitoring tool) can read the file while your application writes to it. Using `FileShare.None` would block all other access.
```

---

## FileInfo (Instance Class)

`FileInfo` wraps a file path into an object with properties and methods. It caches metadata at construction time.

### Construction

```csharp
// Constructor takes a path -- does NOT verify file exists
FileInfo fi = new FileInfo(@"C:\data\report.pdf");
```

### Key Properties

| Property | Type | Description |
|---|---|---|
| `Name` | `string` | File name with extension (e.g., `"report.pdf"`) |
| `FullName` | `string` | Full absolute path |
| `Extension` | `string` | Extension including dot (e.g., `".pdf"`) |
| `Length` | `long` | File size in bytes (throws if file doesn't exist) |
| `DirectoryName` | `string?` | Full path of the parent directory |
| `Directory` | `DirectoryInfo?` | Parent directory as a `DirectoryInfo` object |
| `Exists` | `bool` | Whether the file currently exists on disk |
| `IsReadOnly` | `bool` | Get/set the read-only attribute |
| `CreationTime` | `DateTime` | When the file was created |
| `LastWriteTime` | `DateTime` | When the file was last written to |
| `LastAccessTime` | `DateTime` | When the file was last accessed |
| `Attributes` | `FileAttributes` | File attributes (Hidden, ReadOnly, Archive, etc.) |

### Key Methods

| Method | Return Type | Description |
|---|---|---|
| `CopyTo(dest)` | `FileInfo` | Copy to destination; returns new `FileInfo` for the copy |
| `CopyTo(dest, overwrite)` | `FileInfo` | Copy with optional overwrite |
| `MoveTo(dest)` | `void` | Move/rename the file |
| `Delete()` | `void` | Delete the file |
| `OpenRead()` | `FileStream` | Open for reading |
| `OpenWrite()` | `FileStream` | Open for writing |
| `OpenText()` | `StreamReader` | Open for reading text |
| `CreateText()` | `StreamWriter` | Create for writing text |
| `AppendText()` | `StreamWriter` | Open for appending text |
| `Open(mode)` | `FileStream` | Open with specified `FileMode` |
| `Open(mode, access)` | `FileStream` | Open with mode and access |
| `Open(mode, access, share)` | `FileStream` | Open with full control |
| `Refresh()` | `void` | Re-read cached metadata from disk |

### Example: FileInfo in Action

```csharp
FileInfo fi = new FileInfo(@"C:\data\report.pdf");

if (fi.Exists)
{
    Console.WriteLine($"Name:           {fi.Name}");
    Console.WriteLine($"Full Path:      {fi.FullName}");
    Console.WriteLine($"Extension:      {fi.Extension}");
    Console.WriteLine($"Size:           {fi.Length:N0} bytes");
    Console.WriteLine($"Directory:      {fi.DirectoryName}");
    Console.WriteLine($"Read-Only:      {fi.IsReadOnly}");
    Console.WriteLine($"Created:        {fi.CreationTime}");
    Console.WriteLine($"Last Modified:  {fi.LastWriteTime}");

    // CopyTo returns a FileInfo for the new file
    FileInfo copy = fi.CopyTo(@"C:\backup\report_backup.pdf", overwrite: true);
    Console.WriteLine($"Backed up to:   {copy.FullName}");
}
```

---

## File vs FileInfo -- When to Use Each

| Scenario | Use | Why |
|---|---|---|
| Read a file once | `File.ReadAllText()` | Simplest, no object needed |
| Write a file once | `File.WriteAllText()` | Simplest one-shot write |
| Check if file exists | `File.Exists()` | Static, no allocation |
| Get file size + timestamps + extension | `FileInfo` | One object, multiple properties |
| Loop over files and inspect each | `DirectoryInfo.GetFiles()` -> `FileInfo[]` | Metadata already populated |
| Copy then verify size | `FileInfo.CopyTo()` -> check `Length` | Returns `FileInfo` of the copy |

---

## Practical Example: Common File Operations

```csharp
using System;
using System.IO;

string tempDir = Path.Combine(Path.GetTempPath(), "FileDemo");
Directory.CreateDirectory(tempDir);

try
{
    string filePath = Path.Combine(tempDir, "sample.txt");

    // 1. Write content
    File.WriteAllText(filePath, "Line 1\nLine 2\nLine 3");
    Console.WriteLine("Written: sample.txt");

    // 2. Read it back
    string content = File.ReadAllText(filePath);
    Console.WriteLine($"Content:\n{content}");

    // 3. Append to it
    File.AppendAllText(filePath, "\nLine 4 (appended)");

    // 4. Read as lines
    string[] lines = File.ReadAllLines(filePath);
    Console.WriteLine($"\nLine count: {lines.Length}");

    // 5. Copy it
    string copyPath = Path.Combine(tempDir, "sample_copy.txt");
    File.Copy(filePath, copyPath, overwrite: true);

    // 6. Use FileInfo for metadata comparison
    FileInfo original = new FileInfo(filePath);
    FileInfo copy = new FileInfo(copyPath);
    Console.WriteLine($"\nOriginal: {original.Length} bytes, modified {original.LastWriteTime}");
    Console.WriteLine($"Copy:     {copy.Length} bytes, modified {copy.LastWriteTime}");

    // 7. Move (rename) the copy
    string renamedPath = Path.Combine(tempDir, "sample_renamed.txt");
    File.Move(copyPath, renamedPath);
    Console.WriteLine($"\nRenamed to: {Path.GetFileName(renamedPath)}");
    Console.WriteLine($"Old path exists: {File.Exists(copyPath)}");     // False
    Console.WriteLine($"New path exists: {File.Exists(renamedPath)}");   // True

    // 8. Delete the renamed copy
    File.Delete(renamedPath);
    Console.WriteLine($"Deleted: {File.Exists(renamedPath)}"); // False
}
finally
{
    Directory.Delete(tempDir, recursive: true);
}
```

---

## Summary

- **`File`** (static) is for quick one-shot operations: `ReadAllText`, `WriteAllText`, `Copy`, `Move`, `Delete`, `Exists`.
- **`FileInfo`** (instance) wraps a file path into a reusable object with metadata properties (`Length`, `Extension`, `CreationTime`) and methods that return typed objects.
- `File.ReadAllText()`/`WriteAllText()` are the workhorses for simple file I/O -- they handle open/read/close in one call.
- Prefer `File.ReadLines()` over `ReadAllLines()` for large files -- it reads lazily.
- **`FileMode`** controls create/open/overwrite behavior. **`FileAccess`** controls read/write permissions. **`FileShare`** controls concurrent access by other processes.
- Always use `using` statements when working with streams, readers, and writers obtained from `File.Open*()` or `FileInfo.Open*()` methods.
