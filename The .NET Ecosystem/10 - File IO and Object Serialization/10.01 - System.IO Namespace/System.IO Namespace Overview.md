---
tags:
 - csharp
 - io
 - system-io
---

## System.IO Namespace Overview

The `System.IO` namespace is the primary namespace in .NET for all input/output operations involving the file system, streams, and data serialization to/from bytes and text. Everything from reading a simple text file to monitoring a directory for real-time changes lives here.

At a high level, `System.IO` provides:

- **File and directory manipulation** -- create, delete, copy, move, enumerate
- **Stream-based I/O** -- a unified abstraction for reading/writing byte sequences from any source (files, memory, network, compression, encryption)
- **Text readers/writers** -- character-level I/O layered on top of streams
- **Binary readers/writers** -- reading/writing .NET primitive types in binary format
- **Path utilities** -- string-only path manipulation with no disk I/O
- **Drive information** -- query drive capacity, format, and type
- **File system monitoring** -- real-time event-driven notifications when files change

---

## The Two Design Patterns in System.IO

Almost every file system concept in `System.IO` has **two** representations, and understanding which to use is critical.

### Pattern 1 -- Static Utility Classes

`File`, `Directory`, and `Path` are static classes with static methods. They are designed for **quick, one-shot operations** where you don't need to hold state.

```csharp
// One call, no object allocation beyond the string
string text = File.ReadAllText(@"C:\data\report.txt");

// Check if a directory exists -- single boolean, done
bool exists = Directory.Exists(@"C:\data");

// Build a path -- pure string manipulation
string fullPath = Path.Combine("logs", "2024", "app.log");
```

**When to use**: You need to do one thing and move on. No need to inspect metadata, no need to reuse the reference.

### Pattern 2 -- Instance Classes

`FileInfo`, `DirectoryInfo`, and `DriveInfo` are instance classes you `new` up. They wrap a file system path into a reusable **object** with properties and methods.

```csharp
// Create once, query many properties without repeated path resolution
FileInfo fi = new FileInfo(@"C:\data\report.txt");
Console.WriteLine(fi.Length);        // size in bytes
Console.WriteLine(fi.CreationTime);  // when it was created
Console.WriteLine(fi.Extension);     // ".txt"
Console.WriteLine(fi.DirectoryName); // @"C:\data"
```

**When to use**: You need to perform **multiple operations** on the same file/directory, or you need access to metadata properties (size, timestamps, attributes). Instance classes also return strongly-typed objects from enumeration -- `DirectoryInfo.GetFiles()` returns `FileInfo[]`, not `string[]`.

### Comparison Table

| Aspect | Static (`File`, `Directory`) | Instance (`FileInfo`, `DirectoryInfo`) |
|---|---|---|
| State | No object -- just static method calls | Object wraps a path, caches metadata |
| Return types | Strings and primitives | Typed objects (`FileInfo[]`, `DirectoryInfo[]`) |
| Performance (single op) | Slightly faster -- no object allocation | Slight overhead from object creation |
| Performance (multiple ops) | Each call re-resolves the path | Path resolved once; properties cached |
| Best for | One-shot read/write/check operations | Inspecting metadata, iterating children, reuse |
| Security checks | Performed on every call | Performed once at construction |

```ad-note
title: Caching Caveat
`FileInfo` and `DirectoryInfo` cache some metadata at construction time. If the underlying file changes between construction and property access, call `.Refresh()` to update the cached state.
```

---

## Full Taxonomy of Key Types

| Category | Types | Purpose |
|---|---|---|
| Directory operations | `Directory`, `DirectoryInfo` | Create, delete, enumerate, move directories |
| File operations | `File`, `FileInfo` | Create, delete, copy, move, read/write files |
| Path manipulation | `Path` | Combine paths, get extensions, get temp paths -- **no I/O** |
| Drive information | `DriveInfo` | Query drive space, format, type |
| File system watching | `FileSystemWatcher` | Monitor a directory for changes in real time |
| Streams | `Stream`, `FileStream`, `MemoryStream`, `BufferedStream` | Read/write byte sequences |
| Text readers/writers | `StreamReader`, `StreamWriter`, `StringReader`, `StringWriter` | Read/write character/text data over streams or strings |
| Binary readers/writers | `BinaryReader`, `BinaryWriter` | Read/write .NET primitive types in binary format |
| Exceptions | `IOException`, `FileNotFoundException`, `DirectoryNotFoundException`, `PathTooLongException` | I/O-specific exceptions |

---

## The Stream Hierarchy

**Streams** are the core abstraction in `System.IO`. A stream represents a sequence of bytes that you can read from, write to, or both. The abstract base class `Stream` defines the contract, and concrete subclasses specialize it for different backing stores.

```
Stream (abstract base class)
├── FileStream        -- read/write bytes to/from a file on disk
├── MemoryStream      -- read/write bytes to/from an in-memory byte array
├── BufferedStream    -- wraps another stream, adds a buffer for fewer I/O calls
├── NetworkStream     -- read/write bytes over a network socket (System.Net.Sockets)
├── CryptoStream      -- read/write bytes through a cryptographic transform
├── GZipStream        -- read/write bytes through gzip compression (System.IO.Compression)
├── DeflateStream     -- read/write bytes through deflate compression
├── PipeStream        -- inter-process communication via named/anonymous pipes
└── UnmanagedMemoryStream -- direct access to unmanaged memory blocks
```

Streams operate at the **byte level**. They don't understand text encoding, line breaks, or primitive types -- that's what readers and writers are for.

### Readers and Writers -- Layered on Top of Streams

Readers and writers are **adapters** that sit on top of a `Stream` and provide higher-level operations:

```
┌──────────────────────────────────────────────────────┐
│  StreamReader / StreamWriter                         │
│  (text layer -- reads/writes strings, lines, chars)  │
├──────────────────────────────────────────────────────┤
│  BinaryReader / BinaryWriter                         │
│  (primitive layer -- reads/writes int, double, bool) │
├──────────────────────────────────────────────────────┤
│  Stream (FileStream, MemoryStream, etc.)             │
│  (byte layer -- reads/writes raw byte[])             │
└──────────────────────────────────────────────────────┘
```

```csharp
// StreamWriter wraps a FileStream to write text
using StreamWriter writer = new StreamWriter(
    new FileStream("log.txt", FileMode.Create));
writer.WriteLine("Application started");

// BinaryWriter wraps a FileStream to write primitives
using BinaryWriter bw = new BinaryWriter(
    new FileStream("data.bin", FileMode.Create));
bw.Write(42);          // writes 4 bytes (int)
bw.Write(3.14);        // writes 8 bytes (double)
bw.Write("hello");     // writes length-prefixed UTF-8 string
```

```ad-important
title: Always Dispose Streams
Streams, readers, and writers all implement `IDisposable`. Always wrap them in a `using` statement or `using` declaration. Failing to dispose a writer can cause data loss because buffered data may never get flushed to disk.
```

---

## Common Workflow Patterns

### Quick File I/O (No Streams Needed)

For simple read/write operations, the static `File` class provides convenience methods that handle opening, reading/writing, and closing in a single call:

```csharp
// Read entire file as a single string
string content = File.ReadAllText("config.json");

// Read file as an array of lines
string[] lines = File.ReadAllLines("data.csv");

// Write text -- creates or overwrites
File.WriteAllText("output.txt", "Hello, World!");

// Append text to an existing file
File.AppendAllText("log.txt", $"[{DateTime.Now}] Event occurred\n");
```

### Stream-Based I/O (Fine-Grained Control)

When you need control over buffering, encoding, position, or partial reads:

```csharp
using FileStream fs = new FileStream("data.bin", FileMode.Open, FileAccess.Read);
byte[] buffer = new byte[4096];
int bytesRead;
while ((bytesRead = fs.Read(buffer, 0, buffer.Length)) > 0)
{
    // Process bytesRead bytes from buffer
}
```

### Text Processing with StreamReader/StreamWriter

When you need line-by-line processing or specific text encoding:

```csharp
using StreamReader reader = new StreamReader("data.csv", Encoding.UTF8);
string? line;
while ((line = reader.ReadLine()) is not null)
{
    // Process each line
}
```

---

## Cross-References

| Note | What It Covers |
|---|---|
| [[Directory and DirectoryInfo]] | Creating, listing, moving, deleting directories |
| [[File and FileInfo]] | Reading, writing, copying, moving files; FileMode/FileAccess/FileShare enums |
| [[Path]] | String-based path manipulation -- Combine, GetExtension, GetTempPath, etc. |
| [[DriveInfo]] | Querying drive space, format, type, and readiness |
| [[FileSystemWatcher]] | Monitoring directories for real-time file change events |

---

## Summary

- `System.IO` provides two parallel APIs for file system work: **static classes** (`File`, `Directory`, `Path`) for quick one-shot operations and **instance classes** (`FileInfo`, `DirectoryInfo`) for reusable, metadata-rich objects.
- The **Stream** hierarchy is the core byte-level abstraction. `StreamReader`/`StreamWriter` and `BinaryReader`/`BinaryWriter` are adapters layered on top for text and primitive-type I/O respectively.
- Convenience methods like `File.ReadAllText()` cover 80% of use cases. Streams give you fine-grained control when you need it.
- Always dispose streams, readers, and writers with `using` statements to prevent data loss and resource leaks.
