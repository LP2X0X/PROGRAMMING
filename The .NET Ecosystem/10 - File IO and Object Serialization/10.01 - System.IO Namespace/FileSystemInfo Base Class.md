---
tags:
 - csharp
 - io
 - filesystem
---

## FileSystemInfo -- The Abstract Base for FileInfo and DirectoryInfo

`System.IO.FileSystemInfo` is the **abstract base class** that [[File and FileInfo|FileInfo]] and [[Directory and DirectoryInfo|DirectoryInfo]] both inherit from. It defines the common properties and methods shared by files and directories -- timestamps, attributes, existence checks, names, and paths.

You never instantiate `FileSystemInfo` directly. Instead, you work with its concrete subclasses. However, understanding the base class is important because it defines the shared API, and you can write polymorphic code that accepts `FileSystemInfo` to work with both files and directories.

---

## Inheritance Hierarchy

```
System.MarshalByRefObject
  └── FileSystemInfo (abstract)
        ├── FileInfo
        └── DirectoryInfo
```

Any property or method on `FileSystemInfo` is available on both `FileInfo` and `DirectoryInfo`.

---

## Shared Properties

These properties are inherited by both `FileInfo` and `DirectoryInfo`:

### Name and Path Properties

| Property | Type | Description |
|---|---|---|
| `Name` | `string` | The name of the file or directory. For `FileInfo`, this is the file name with extension (e.g., `"data.txt"`). For `DirectoryInfo`, this is the directory name only (e.g., `"Documents"`). |
| `FullName` | `string` | The complete absolute path (e.g., `"C:\Users\admin\Documents\data.txt"`) |
| `Extension` | `string` | The file extension including the dot (e.g., `".txt"`). For directories, this is typically empty unless the directory name contains a dot. |

### Existence

| Property | Type | Description |
|---|---|---|
| `Exists` | `bool` | `true` if the file or directory exists on disk. This is an **abstract** property -- `FileInfo` and `DirectoryInfo` each implement it. |

### Timestamps

| Property | Type | Description |
|---|---|---|
| `CreationTime` | `DateTime` | When the file or directory was created (local time) |
| `CreationTimeUtc` | `DateTime` | When created (UTC) |
| `LastAccessTime` | `DateTime` | When last accessed (local time) |
| `LastAccessTimeUtc` | `DateTime` | When last accessed (UTC) |
| `LastWriteTime` | `DateTime` | When last modified (local time) |
| `LastWriteTimeUtc` | `DateTime` | When last modified (UTC) |

```ad-note
title: Prefer UTC Timestamps
When comparing or storing timestamps, always use the `Utc` variants (`CreationTimeUtc`, `LastWriteTimeUtc`, etc.). Local-time versions depend on the machine's time zone, which can cause subtle bugs on servers, in CI/CD pipelines, or when transferring data between machines.
```

### Attributes

| Property | Type | Description |
|---|---|---|
| `Attributes` | `FileAttributes` | A flags enum describing the file/directory: `ReadOnly`, `Hidden`, `System`, `Directory`, `Archive`, `Normal`, `Temporary`, etc. |

```csharp
var fi = new FileInfo(@"C:\data\config.json");

// Check individual flags
bool isReadOnly = (fi.Attributes & FileAttributes.ReadOnly) != 0;
bool isHidden   = fi.Attributes.HasFlag(FileAttributes.Hidden);

// Set a flag
fi.Attributes |= FileAttributes.ReadOnly;

// Clear a flag
fi.Attributes &= ~FileAttributes.ReadOnly;
```

### Link Target (.NET 6+)

| Property | Type | Description |
|---|---|---|
| `LinkTarget` | `string?` | If this file or directory is a symbolic link, returns the target path. Otherwise `null`. |

---

## Shared Methods

| Method | Description |
|---|---|
| `Delete()` | Deletes the file or directory. Abstract -- each subclass implements its own behavior. `DirectoryInfo.Delete()` only deletes an empty directory by default; use `Delete(true)` for recursive delete. |
| `Refresh()` | Re-reads all metadata from disk. Clears the cached property values and forces the next property access to read fresh data from the file system. |
| `GetObjectData()` | Serialization support (rarely used directly). |

---

## The Caching Behavior -- Why Refresh() Matters

This is the most important thing to understand about `FileSystemInfo` and its subclasses: ==property values are cached at construction time and do NOT automatically update when the file system changes==.

When you create a `FileInfo` or `DirectoryInfo`, the runtime queries the file system once and caches the metadata (size, timestamps, attributes, existence). After that, reading properties returns the **cached** values, even if the file has changed on disk.

```csharp
var fi = new FileInfo("data.txt");

Console.WriteLine(fi.Length);     // reads from disk, then caches the value

// ... another process or thread modifies data.txt ...

Console.WriteLine(fi.Length);     // STILL returns the old cached value!

fi.Refresh();                     // re-query the file system

Console.WriteLine(fi.Length);     // NOW returns the updated size
```

```ad-warning
title: Stale Properties Can Cause Bugs
If your code creates a `FileInfo`/`DirectoryInfo` object and then later checks `Exists`, `Length`, or timestamps, the values may be stale. This is especially dangerous in multi-threaded applications, file watchers, or long-running processes. Always call `Refresh()` before reading properties if the file might have changed since the object was created.
```

The caching behavior applies to **all** properties inherited from `FileSystemInfo`:
- `Exists`
- `Length` (on `FileInfo`)
- `CreationTime` / `CreationTimeUtc`
- `LastAccessTime` / `LastAccessTimeUtc`
- `LastWriteTime` / `LastWriteTimeUtc`
- `Attributes`

```csharp
var fi = new FileInfo("new_file.txt");
Console.WriteLine(fi.Exists);    // False (file doesn't exist yet)

File.WriteAllText("new_file.txt", "content");

Console.WriteLine(fi.Exists);    // STILL False -- cached!
fi.Refresh();
Console.WriteLine(fi.Exists);    // True -- refreshed from disk
```

```ad-note
title: Why Does .NET Cache?
Querying the file system is a relatively expensive operation (it involves a kernel call). Caching avoids repeated OS calls when you access multiple properties of the same file in sequence. The tradeoff is that you must explicitly call `Refresh()` if the file might have changed since construction.
```

---

## FileAttributes Flags Enum

The `Attributes` property returns a `FileAttributes` flags enum. Multiple flags can be combined.

| Flag | Value | Description |
|---|---|---|
| `ReadOnly` | 1 | File is read-only |
| `Hidden` | 2 | File is hidden in directory listings |
| `System` | 4 | File is a system file |
| `Directory` | 16 | Entry is a directory |
| `Archive` | 32 | File is a candidate for backup (set by most write operations) |
| `Normal` | 128 | File has no other attributes set. Only valid when used alone. |
| `Temporary` | 256 | File is temporary -- the file system may keep it in memory |
| `SparseFile` | 512 | File is a sparse file (NTFS) |
| `ReparsePoint` | 1024 | File contains a reparse point (symbolic link, junction, etc.) |
| `Compressed` | 2048 | File is compressed (NTFS attribute, not .zip) |
| `Encrypted` | 16384 | File is encrypted (EFS on Windows) |

```csharp
var fi = new FileInfo("report.pdf");

// Check if hidden
if (fi.Attributes.HasFlag(FileAttributes.Hidden))
    Console.WriteLine("File is hidden");

// Check if it's a symbolic link
if (fi.Attributes.HasFlag(FileAttributes.ReparsePoint))
    Console.WriteLine($"Symlink target: {fi.LinkTarget}");
```

---

## Polymorphism with FileSystemInfo

Because `FileInfo` and `DirectoryInfo` share a common base, you can write code that handles both:

```csharp
void PrintInfo(FileSystemInfo fsi)
{
    Console.WriteLine($"Name:         {fsi.Name}");
    Console.WriteLine($"Full Path:    {fsi.FullName}");
    Console.WriteLine($"Exists:       {fsi.Exists}");
    Console.WriteLine($"Last Modified: {fsi.LastWriteTime}");
    Console.WriteLine($"Attributes:   {fsi.Attributes}");

    if (fsi is FileInfo file)
        Console.WriteLine($"Size:         {file.Length} bytes");
}

// Works with both:
PrintInfo(new FileInfo(@"C:\data\report.pdf"));
PrintInfo(new DirectoryInfo(@"C:\data"));
```

This is especially useful when working with `DirectoryInfo.GetFileSystemInfos()`, which returns a mixed array of `FileInfo` and `DirectoryInfo` objects:

```csharp
var dir = new DirectoryInfo(@"C:\Projects");

// Get all files AND directories in one call
FileSystemInfo[] entries = dir.GetFileSystemInfos();

foreach (FileSystemInfo entry in entries)
{
    string type = entry is DirectoryInfo ? "DIR " : "FILE";
    Console.WriteLine($"[{type}] {entry.Name,-30} {entry.LastWriteTime:yyyy-MM-dd}");
}
```

---

## Summary

| Concept | Detail |
|---|---|
| What it is | Abstract base class for `FileInfo` and `DirectoryInfo` |
| Shared properties | `Name`, `FullName`, `Extension`, `Exists`, `CreationTime(Utc)`, `LastAccessTime(Utc)`, `LastWriteTime(Utc)`, `Attributes`, `LinkTarget` |
| Shared methods | `Delete()`, `Refresh()` |
| Caching behavior | Properties are cached at construction time -- call `Refresh()` to re-read from disk |
| `FileAttributes` | Flags enum: `ReadOnly`, `Hidden`, `System`, `Directory`, `Archive`, `Normal`, `Temporary`, etc. |
| Polymorphism | Write code that accepts `FileSystemInfo` to handle both files and directories generically |
