---
tags:
 - csharp
 - io
 - paths
---

## Path

`Path` is a static utility class for **string-based path manipulation**. It provides methods to combine, parse, and transform file paths without ever touching the disk. No file or directory needs to exist for any `Path` method to work -- it operates purely on strings.

This makes `Path` fast, safe, and free of I/O exceptions, but it also means `Path` will happily return results for nonsensical or nonexistent paths.

---

## Combining Paths

### Path.Combine -- The Right Way to Build Paths

```csharp
// Correctly handles directory separators -- don't ever concatenate manually
string path = Path.Combine("C:", "Users", "admin", "Documents", "report.txt");
// Result: "C:\Users\admin\Documents\report.txt"

// Handles trailing separators gracefully
string path = Path.Combine(@"C:\Users\", "admin", "file.txt");
// Result: "C:\Users\admin\file.txt"

// Accepts 2, 3, 4, or params string[] arguments
string path = Path.Combine("folder", "subfolder", "file.txt");
```

```ad-warning
title: NEVER Concatenate Paths Manually
Do NOT do this:

    string path = folder + "\\" + filename;
    string path = $"{folder}\\{filename}";

This breaks on Linux/macOS (which use `/`), double-separators appear if `folder` already ends with `\`, and you lose edge case handling. Always use `Path.Combine()`.
```

```ad-warning
title: Path.Combine Gotcha with Rooted Paths
If any argument after the first is an absolute (rooted) path, `Path.Combine` discards everything before it:

    Path.Combine(@"C:\Users", @"D:\Data", "file.txt")
    // Result: "D:\Data\file.txt"  -- C:\Users is silently DROPPED

    Path.Combine("folder", "/absolute", "file.txt")
    // Result: "/absolute\file.txt"  -- "folder" is silently DROPPED

This is by design but catches people off guard. If paths come from user input, validate them first.
```

### Path.Join -- .NET Core 2.1+ Alternative

`Path.Join()` is similar to `Combine` but does **not** discard earlier segments when it encounters a rooted path. It simply concatenates with a separator.

```csharp
// Path.Combine drops earlier segments if a later one is rooted
Path.Combine(@"C:\Base", @"D:\Other")  // "D:\Other"

// Path.Join just concatenates -- no dropping
Path.Join(@"C:\Base", @"D:\Other")     // "C:\Base\D:\Other"

// Path.Join also works with ReadOnlySpan<char> for zero-allocation path building
```

```ad-note
title: Combine vs Join
- Use `Path.Combine()` for most cases -- it's the standard, well-known approach.
- Use `Path.Join()` when you want strict concatenation without segment-dropping behavior, or when working with spans for performance.
```

---

## Parsing Path Components

`Path` can decompose any path string into its constituent parts.

```csharp
string path = @"C:\Projects\MyApp\src\Program.cs";

Path.GetFileName(path)                  // "Program.cs"
Path.GetFileNameWithoutExtension(path)  // "Program"
Path.GetExtension(path)                 // ".cs"  (includes the dot)
Path.GetDirectoryName(path)             // @"C:\Projects\MyApp\src"
Path.GetPathRoot(path)                  // @"C:\"
```

### Complete Decomposition Example

```csharp
string path = @"C:\Projects\MyApp\src\Program.cs";

Console.WriteLine($"Full path:      {path}");
Console.WriteLine($"Root:           {Path.GetPathRoot(path)}");         // C:\
Console.WriteLine($"Directory:      {Path.GetDirectoryName(path)}");    // C:\Projects\MyApp\src
Console.WriteLine($"File name:      {Path.GetFileName(path)}");         // Program.cs
Console.WriteLine($"Name only:      {Path.GetFileNameWithoutExtension(path)}"); // Program
Console.WriteLine($"Extension:      {Path.GetExtension(path)}");       // .cs
Console.WriteLine($"Is rooted:      {Path.IsPathRooted(path)}");       // True
Console.WriteLine($"Has extension:  {Path.HasExtension(path)}");       // True
```

### Edge Cases

```csharp
// Directory paths (no file)
Path.GetFileName(@"C:\Projects\MyApp\")          // "" (empty string)
Path.GetDirectoryName(@"C:\Projects\MyApp\")     // "C:\Projects\MyApp"

// File name with no directory
Path.GetDirectoryName("file.txt")                // "" (empty string)
Path.GetFileName("file.txt")                     // "file.txt"

// No extension
Path.GetExtension("Makefile")                    // "" (empty string)
Path.HasExtension("Makefile")                    // false

// Multiple dots
Path.GetExtension("archive.tar.gz")             // ".gz" (last extension only)
Path.GetFileNameWithoutExtension("archive.tar.gz") // "archive.tar"

// Null input
Path.GetFileName(null)                           // null (doesn't throw)
Path.GetExtension(null)                          // null
```

---

## Resolving and Transforming Paths

### Path.GetFullPath -- Resolve Relative Paths

```csharp
// Resolves relative paths against the current working directory
string full = Path.GetFullPath("data.txt");
// If CWD is C:\Projects -> "C:\Projects\data.txt"

string full = Path.GetFullPath(@"..\other\file.txt");
// If CWD is C:\Projects\MyApp -> "C:\Projects\other\file.txt"

// Absolute paths pass through unchanged
string full = Path.GetFullPath(@"C:\absolute\path.txt");
// "C:\absolute\path.txt"

// .NET Core 2.1+: Resolve relative to a specific base path
string full = Path.GetFullPath("file.txt", @"D:\Base");
// "D:\Base\file.txt"
```

### Path.GetRelativePath -- .NET Core 2.0+

```csharp
// Get a relative path from one location to another
string relative = Path.GetRelativePath(
    @"C:\Projects\MyApp",           // base path
    @"C:\Projects\MyApp\src\file.cs" // target path
);
// Result: "src\file.cs"

string relative = Path.GetRelativePath(
    @"C:\Projects\MyApp\src",
    @"C:\Projects\Shared\utils.cs"
);
// Result: "..\..\Shared\utils.cs"
```

### Path.ChangeExtension

```csharp
// Change the extension of a file path (string operation only -- no file renamed)
Path.ChangeExtension("report.docx", ".pdf")     // "report.pdf"
Path.ChangeExtension("archive.tar.gz", ".zip")   // "archive.tar.zip"
Path.ChangeExtension("report.docx", null)         // "report"  (removes extension)
Path.ChangeExtension("Makefile", ".bak")          // "Makefile.bak"
```

---

## Temporary Files and Paths

### Path.GetTempPath

```csharp
// Returns the system temp directory path
string tempDir = Path.GetTempPath();
// Typically: C:\Users\<username>\AppData\Local\Temp\
```

### Path.GetTempFileName

```csharp
// Creates a ZERO-BYTE file in the temp directory and returns its path
string tempFile = Path.GetTempFileName();
// Example: "C:\Users\admin\AppData\Local\Temp\tmp1A2B.tmp"
```

```ad-warning
title: GetTempFileName Actually Creates a File
Unlike most `Path` methods, `GetTempFileName()` performs I/O -- it creates an actual empty file on disk. You must clean it up when done. Also, Windows limits the temp directory to ~65,535 temp files created this way. If you don't clean up, you'll eventually hit an `IOException`.
```

### Path.GetRandomFileName

```csharp
// Generates a random file name string -- does NOT create any file
string randomName = Path.GetRandomFileName();
// Example: "xkr5zpb0.kgv"  (8.3 format, cryptographically random)

// Useful for creating unique directories or files yourself
string uniqueDir = Path.Combine(Path.GetTempPath(), Path.GetRandomFileName());
Directory.CreateDirectory(uniqueDir);
```

```ad-note
title: GetTempFileName vs GetRandomFileName
- `GetTempFileName()`: creates a real file, returns its path, has a file count limit.
- `GetRandomFileName()`: returns a random string only, creates nothing, no limits.

For most cases, prefer `GetRandomFileName()` combined with `Path.Combine()` and manual file creation. It gives you more control and doesn't have the file count limitation.
```

---

## Platform-Specific Constants

`Path` exposes platform-dependent constants so you never need to hardcode separators:

```csharp
// Directory separator for the current OS
char sep = Path.DirectorySeparatorChar;
// Windows: '\\'
// Linux/macOS: '/'

// Alternate directory separator (Windows accepts both \ and /)
char altSep = Path.AltDirectorySeparatorChar;
// Windows: '/'
// Linux/macOS: '/'

// Path separator for PATH environment variable
char pathSep = Path.PathSeparator;
// Windows: ';'  (e.g., C:\bin;C:\tools)
// Linux/macOS: ':'  (e.g., /usr/bin:/usr/local/bin)

// Volume separator
char volSep = Path.VolumeSeparatorChar;
// Windows: ':'  (as in C:)
// Linux: '/'

// Characters not allowed in file names
char[] invalidFileChars = Path.GetInvalidFileNameChars();

// Characters not allowed in paths
char[] invalidPathChars = Path.GetInvalidPathChars();
```

### Validating Paths

```csharp
// Check if a file name contains invalid characters
string fileName = "report<v2>.txt";
bool isValid = fileName.IndexOfAny(Path.GetInvalidFileNameChars()) == -1;
// false -- '<' and '>' are invalid

// Strip invalid characters from a file name
string sanitized = string.Concat(
    fileName.Where(c => !Path.GetInvalidFileNameChars().Contains(c))
);
// "reportv2.txt"
```

---

## Path.IsPathRooted and Path.HasExtension

These are simple boolean checks, useful for input validation:

```csharp
// Is the path absolute (rooted)?
Path.IsPathRooted(@"C:\file.txt")   // true
Path.IsPathRooted(@"\file.txt")     // true (rooted but no drive)
Path.IsPathRooted("file.txt")       // false (relative)

// Does the path end with a file extension?
Path.HasExtension("file.txt")       // true
Path.HasExtension("Makefile")       // false
Path.HasExtension(".gitignore")     // true (.gitignore is treated as extension)
```

---

## .NET 6+ Path Additions

```csharp
// Path.Exists -- checks if a file OR directory exists (.NET 7+)
bool exists = Path.Exists(@"C:\somepath");  // true for file or directory

// Path.TrimEndingDirectorySeparator -- removes trailing \ or /
Path.TrimEndingDirectorySeparator(@"C:\folder\")  // "C:\folder"

// Path.EndsInDirectorySeparator -- check for trailing separator
Path.EndsInDirectorySeparator(@"C:\folder\")  // true
Path.EndsInDirectorySeparator(@"C:\file.txt")  // false
```

---

## Practical Example: Path Parsing Utility

```csharp
using System;
using System.IO;

void AnalyzePath(string path)
{
    Console.WriteLine($"Input:          \"{path}\"");
    Console.WriteLine($"  Root:         \"{Path.GetPathRoot(path)}\"");
    Console.WriteLine($"  Directory:    \"{Path.GetDirectoryName(path)}\"");
    Console.WriteLine($"  File Name:    \"{Path.GetFileName(path)}\"");
    Console.WriteLine($"  Name Only:    \"{Path.GetFileNameWithoutExtension(path)}\"");
    Console.WriteLine($"  Extension:    \"{Path.GetExtension(path)}\"");
    Console.WriteLine($"  Is Rooted:    {Path.IsPathRooted(path)}");
    Console.WriteLine($"  Has Ext:      {Path.HasExtension(path)}");
    Console.WriteLine($"  Full Path:    \"{Path.GetFullPath(path)}\"");
    Console.WriteLine();
}

// Test with various paths
AnalyzePath(@"C:\Projects\MyApp\src\Program.cs");
AnalyzePath("relative/path/data.csv");
AnalyzePath(@"..\sibling\file.txt");
AnalyzePath("Makefile");
AnalyzePath(@"C:\Logs\");
```

Output (assuming CWD is `C:\Projects\MyApp`):

```
Input:          "C:\Projects\MyApp\src\Program.cs"
  Root:         "C:\"
  Directory:    "C:\Projects\MyApp\src"
  File Name:    "Program.cs"
  Name Only:    "Program"
  Extension:    ".cs"
  Is Rooted:    True
  Has Ext:      True
  Full Path:    "C:\Projects\MyApp\src\Program.cs"

Input:          "relative/path/data.csv"
  Root:         ""
  Directory:    "relative/path"
  File Name:    "data.csv"
  Name Only:    "data"
  Extension:    ".csv"
  Is Rooted:    False
  Has Ext:      True
  Full Path:    "C:\Projects\MyApp\relative\path\data.csv"
```

---

## Summary

- `Path` is a **pure string manipulation** class -- it performs no disk I/O (except `GetTempFileName()` which is the one exception).
- **Always** use `Path.Combine()` to build paths -- never concatenate with `+` or string interpolation.
- Watch out for `Path.Combine()` dropping earlier segments when a later argument is rooted.
- Use `Path.GetFullPath()` to resolve relative paths to absolute.
- Use `Path.GetRelativePath()` (.NET Core 2.0+) to compute relative paths between two locations.
- `GetTempFileName()` creates a real file and has a count limit; prefer `GetRandomFileName()` + manual creation.
- Use `Path.GetInvalidFileNameChars()` and `Path.GetInvalidPathChars()` for input validation.
- `Path.DirectorySeparatorChar` and friends let you write platform-independent code.
