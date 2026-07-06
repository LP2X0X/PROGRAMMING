---
tags:
 - csharp
 - io
 - streams
---

## FileStream -- Byte-Level File I/O

`FileStream` is the primary [[Stream]] subclass for reading and writing **bytes** to and from files on disk. It is the lowest-level file I/O class in `System.IO` -- everything else (`File.ReadAllText`, `StreamReader`, etc.) ultimately uses a `FileStream` under the hood.

Use `FileStream` when you need:

- **Chunked reading** of large files without loading them entirely into memory
- **Precise control** over file access mode, sharing, and buffering
- **Seeking** (random access) to specific positions within a file
- **True asynchronous I/O** via the OS kernel
- **Byte-level operations** on binary files (images, executables, custom formats)

For higher-level text operations, wrap a `FileStream` with a [[StreamReader and StreamWriter|StreamReader or StreamWriter]]. For primitive type serialization, use [[BinaryReader and BinaryWriter]].

---

## Creating a FileStream

### Direct Construction

The `FileStream` constructor accepts several parameters that control how the file is opened, accessed, and shared.

```csharp
public FileStream(
    string path,               // file path
    FileMode mode,             // how to open the file
    FileAccess access,         // read, write, or both
    FileShare share,           // how other processes can access the file concurrently
    int bufferSize,            // internal buffer size in bytes (default 4096)
    bool useAsync              // true for asynchronous I/O at the OS level
)
```

### FileMode Enum

Controls **how** the file is opened or created.

| Value | File Exists | File Does Not Exist | Description |
|---|---|---|---|
| `FileMode.Create` | Overwrites (truncates to 0) | Creates new | Create or overwrite |
| `FileMode.CreateNew` | Throws `IOException` | Creates new | Create only -- fail if exists |
| `FileMode.Open` | Opens existing | Throws `FileNotFoundException` | Open only -- fail if not exists |
| `FileMode.OpenOrCreate` | Opens existing | Creates new | Open or create -- never fails |
| `FileMode.Append` | Opens, seeks to end | Creates new | Append-only writing |
| `FileMode.Truncate` | Opens, truncates to 0 | Throws `FileNotFoundException` | Open and empty the file |

### FileAccess Enum

Controls **what operations** are allowed.

| Value | Description |
|---|---|
| `FileAccess.Read` | Read-only. `Write()` throws `NotSupportedException`. |
| `FileAccess.Write` | Write-only. `Read()` throws `NotSupportedException`. |
| `FileAccess.ReadWrite` | Both reading and writing. |

### FileShare Enum

Controls how **other processes** (or other `FileStream` instances) can access the file while this one has it open. This is critical for multi-process scenarios.

| Value | Description |
|---|---|
| `FileShare.None` | Exclusive lock -- no other process can open the file at all. |
| `FileShare.Read` | Other processes can open the file for reading but not writing. |
| `FileShare.Write` | Other processes can open the file for writing but not reading. |
| `FileShare.ReadWrite` | Other processes can open the file for both reading and writing. |
| `FileShare.Delete` | Other processes can delete the file while it is open. |

```ad-warning
title: FileShare.None Can Cause Contention
Using `FileShare.None` prevents all other access to the file. If another process (or thread) tries to open the file, it gets an `IOException`. Use the least restrictive sharing mode that meets your needs. For read-only operations, `FileShare.Read` is usually appropriate.
```

### Common Constructor Patterns

```csharp
// Read a file -- others can also read it
var fs1 = new FileStream("data.bin", FileMode.Open, FileAccess.Read, FileShare.Read);

// Create a new file for exclusive writing
var fs2 = new FileStream("output.bin", FileMode.Create, FileAccess.Write, FileShare.None);

// Open or create for read/write -- shared reading allowed
var fs3 = new FileStream("db.dat", FileMode.OpenOrCreate, FileAccess.ReadWrite, FileShare.Read);

// Async I/O -- critical for true async behavior
var fs4 = new FileStream("data.bin", FileMode.Open, FileAccess.Read,
    FileShare.Read, bufferSize: 4096, useAsync: true);
```

### Via File Static Methods

The `File` class provides convenience methods that return pre-configured `FileStream` instances:

```csharp
// File.Open -- most flexible convenience method
FileStream fs1 = File.Open("data.bin", FileMode.Open);
FileStream fs2 = File.Open("data.bin", FileMode.Open, FileAccess.Read, FileShare.Read);

// File.OpenRead -- read-only, FileShare.Read
// Equivalent to: new FileStream(path, FileMode.Open, FileAccess.Read, FileShare.Read)
FileStream fs3 = File.OpenRead("data.bin");

// File.OpenWrite -- write-only, opens or creates
// Equivalent to: new FileStream(path, FileMode.OpenOrCreate, FileAccess.Write, FileShare.None)
FileStream fs4 = File.OpenWrite("data.bin");

// File.Create -- create or overwrite, read/write
// Equivalent to: new FileStream(path, FileMode.Create, FileAccess.ReadWrite, FileShare.None)
FileStream fs5 = File.Create("data.bin");
```

```ad-note
title: File.OpenWrite Does NOT Truncate
`File.OpenWrite()` uses `FileMode.OpenOrCreate`, which means if the file already exists and has 1000 bytes, opening and writing 100 bytes leaves the remaining 900 bytes **intact**. If you want to overwrite completely, use `File.Create()` instead, which uses `FileMode.Create` (truncates first).
```

---

## Reading Bytes

### Chunked Reading (Large Files)

The standard pattern for reading a file in chunks without loading it entirely into memory:

```csharp
using var fs = File.OpenRead("photo.jpg");
byte[] buffer = new byte[4096]; // 4 KB buffer
int bytesRead;

while ((bytesRead = fs.Read(buffer, 0, buffer.Length)) > 0)
{
    // Only buffer[0..bytesRead-1] contains valid data
    ProcessChunk(buffer, bytesRead);
}
```

```ad-important
title: Read() Returns Fewer Bytes Than Requested
`Read()` may return fewer bytes than you asked for, even when more data is available. The return value tells you how many bytes were actually read. ==Always use the return value, never assume the buffer is full.== A return of `0` means end-of-stream.
```

### Reading Entire File into byte[]

For small files where loading everything into memory is acceptable:

```csharp
// Option 1: File.ReadAllBytes (simplest)
byte[] allBytes = File.ReadAllBytes("small.bin");

// Option 2: Via FileStream + MemoryStream
using var fs = File.OpenRead("small.bin");
using var ms = new MemoryStream();
fs.CopyTo(ms);
byte[] allBytes2 = ms.ToArray();
```

### Reading a Single Byte

```csharp
using var fs = File.OpenRead("data.bin");
int b = fs.ReadByte(); // returns 0-255, or -1 at end-of-stream
if (b != -1)
{
    byte value = (byte)b;
}
```

---

## Writing Bytes

### Writing a Byte Array

```csharp
using var fs = new FileStream("output.bin", FileMode.Create, FileAccess.Write);
byte[] data = { 0x48, 0x65, 0x6C, 0x6C, 0x6F }; // "Hello" in ASCII
fs.Write(data, 0, data.Length);
```

### Writing a Single Byte

```csharp
using var fs = new FileStream("output.bin", FileMode.Create);
fs.WriteByte(0xFF);
fs.WriteByte(0x00);
fs.WriteByte(0xAB);
```

### Building a File from Multiple Chunks

```csharp
using var fs = new FileStream("combined.bin", FileMode.Create, FileAccess.Write);

byte[] header = { 0x89, 0x50, 0x4E, 0x47 }; // PNG magic bytes
fs.Write(header, 0, header.Length);

byte[] body = GetBodyData();
fs.Write(body, 0, body.Length);

byte[] footer = GetFooterData();
fs.Write(footer, 0, footer.Length);
// Dispose flushes remaining buffer to disk
```

---

## Seeking -- Random Access

`FileStream` supports seeking (`CanSeek == true`), which lets you jump to any byte position in the file.

```csharp
using var fs = new FileStream("records.bin", FileMode.Open, FileAccess.ReadWrite);

// Jump to the beginning
fs.Seek(0, SeekOrigin.Begin);

// Jump to byte 1000
fs.Seek(1000, SeekOrigin.Begin);

// Jump 10 bytes before the end
fs.Seek(-10, SeekOrigin.End);

// Move 50 bytes forward from current position
fs.Seek(50, SeekOrigin.Current);

// Move 20 bytes backward from current position
fs.Seek(-20, SeekOrigin.Current);

// You can also set Position directly (equivalent to Seek from Begin)
fs.Position = 500;
```

### Practical Example -- Updating a Record in Place

```csharp
// Fixed-size records: each record is exactly 64 bytes
const int RecordSize = 64;

using var fs = new FileStream("database.bin", FileMode.Open, FileAccess.ReadWrite);

// Jump to record #10
fs.Seek(10 * RecordSize, SeekOrigin.Begin);

// Read the record
byte[] record = new byte[RecordSize];
fs.Read(record, 0, RecordSize);

// Modify the record
record[0] = 0x01; // set a flag byte

// Seek BACK to the start of record #10 and overwrite
fs.Seek(10 * RecordSize, SeekOrigin.Begin);
fs.Write(record, 0, RecordSize);
```

---

## Asynchronous File I/O

### The useAsync Constructor Parameter

```ad-important
title: useAsync Is Required for True Async I/O
When you call `ReadAsync()` or `WriteAsync()` on a `FileStream`, the behavior depends on whether the stream was opened with `useAsync: true`:

- **`useAsync: true`** -- The OS performs the I/O asynchronously using I/O completion ports (Windows) or `epoll`/`kqueue` (Linux/macOS). The calling thread is **freed** while the I/O is in progress. This is true non-blocking I/O.
- **`useAsync: false` (default)** -- `ReadAsync`/`WriteAsync` wrap the synchronous `Read`/`Write` on a thread pool thread. The thread pool thread **blocks** waiting for the I/O. You get the `async`/`await` syntax, but you are consuming a thread pool thread for the duration of the I/O.

==Always set `useAsync: true` when you intend to use async methods.==
```

```csharp
// Correct: true async I/O
await using var fs = new FileStream("data.bin", FileMode.Open, FileAccess.Read,
    FileShare.Read, bufferSize: 4096, useAsync: true);

byte[] buffer = new byte[4096];
int bytesRead = await fs.ReadAsync(buffer, 0, buffer.Length);
```

### Async Reading Loop

```csharp
await using var fs = new FileStream("large.bin", FileMode.Open, FileAccess.Read,
    FileShare.Read, bufferSize: 4096, useAsync: true);

byte[] buffer = new byte[81920]; // 80 KB buffer for large file reads
int bytesRead;

while ((bytesRead = await fs.ReadAsync(buffer, 0, buffer.Length)) > 0)
{
    await ProcessChunkAsync(buffer, bytesRead);
}
```

### Async Writing

```csharp
await using var fs = new FileStream("output.bin", FileMode.Create, FileAccess.Write,
    FileShare.None, bufferSize: 4096, useAsync: true);

byte[] data = GenerateData();
await fs.WriteAsync(data, 0, data.Length);
await fs.FlushAsync(); // ensure data is committed to disk
```

### Async with CancellationToken

```csharp
async Task ReadFileAsync(string path, CancellationToken ct)
{
    await using var fs = new FileStream(path, FileMode.Open, FileAccess.Read,
        FileShare.Read, bufferSize: 4096, useAsync: true);

    byte[] buffer = new byte[4096];
    int bytesRead;

    while ((bytesRead = await fs.ReadAsync(buffer, 0, buffer.Length, ct)) > 0)
    {
        // Process chunk
        // If ct is cancelled, ReadAsync throws OperationCanceledException
    }
}
```

---

## FileStream and IDisposable

`FileStream` holds an **OS file handle** -- a limited, unmanaged resource. Failing to dispose a `FileStream` means:

- The file handle leaks until the garbage collector eventually finalizes the object (nondeterministic)
- Other processes may be locked out of the file due to `FileShare` restrictions
- Buffered data may not be flushed, causing data loss on writes

```csharp
// ALWAYS use 'using' to guarantee disposal
using var fs = new FileStream("data.bin", FileMode.Create);
fs.Write(data, 0, data.Length);
// fs is disposed here -- file handle released, buffer flushed

// For async code in C# 8+:
await using var fs2 = new FileStream("data.bin", FileMode.Create,
    FileAccess.Write, FileShare.None, 4096, useAsync: true);
await fs2.WriteAsync(data, 0, data.Length);
// DisposeAsync called here
```

---

## Buffer Size

The `bufferSize` parameter controls the size of the internal read/write buffer. Bytes are accumulated in this buffer and written to disk in larger chunks, reducing the number of OS I/O calls.

| Buffer Size | Behavior |
|---|---|
| Default (4096) | Good general-purpose choice |
| Larger (e.g., 81920) | Better throughput for sequential reads/writes on large files |
| Smaller (e.g., 512) | Lower memory overhead for many simultaneous file handles |
| 1 | Effectively unbuffered -- every read/write goes directly to the OS |

```csharp
// Large buffer for bulk sequential I/O
var fs = new FileStream("bigfile.dat", FileMode.Open, FileAccess.Read,
    FileShare.Read, bufferSize: 81920);
```

```ad-note
title: Buffer Size vs Your Read Buffer
The `bufferSize` in the constructor is the **internal** buffer `FileStream` uses for OS calls. Your `byte[] buffer` in the `Read()` call is a **separate** application-level buffer. Both exist simultaneously. For maximum performance on large sequential reads, make both large (e.g., 80 KB+).
```

---

## FileStream vs File Convenience Methods

| Scenario | Recommended Approach |
|---|---|
| Read entire small file as text | `File.ReadAllText()` |
| Read entire small file as bytes | `File.ReadAllBytes()` |
| Read file line by line | `File.ReadLines()` or [[StreamReader and StreamWriter\|StreamReader]] |
| Write a string to a file | `File.WriteAllText()` |
| Append text | `File.AppendAllText()` |
| Large file -- process in chunks | `FileStream` with a read loop |
| Need seeking / random access | `FileStream` with `Seek()` |
| Need file sharing control | `FileStream` with `FileShare` parameter |
| Async I/O required | `FileStream` with `useAsync: true` |
| Binary primitives (int, double) | [[BinaryReader and BinaryWriter]] wrapping a `FileStream` |
| Text with specific encoding | [[StreamReader and StreamWriter]] wrapping a `FileStream` |

---

## Practical Example -- File Copy with Progress Reporting

A real-world example combining reading, writing, seeking (`Length`), and progress tracking:

```csharp
public static async Task CopyWithProgressAsync(
    string sourcePath,
    string destPath,
    IProgress<double> progress,
    CancellationToken ct = default)
{
    await using var source = new FileStream(sourcePath, FileMode.Open, FileAccess.Read,
        FileShare.Read, bufferSize: 81920, useAsync: true);
    await using var dest = new FileStream(destPath, FileMode.Create, FileAccess.Write,
        FileShare.None, bufferSize: 81920, useAsync: true);

    long totalBytes = source.Length;
    long totalRead = 0;
    byte[] buffer = new byte[81920];
    int bytesRead;

    while ((bytesRead = await source.ReadAsync(buffer, 0, buffer.Length, ct)) > 0)
    {
        await dest.WriteAsync(buffer, 0, bytesRead, ct);
        totalRead += bytesRead;
        progress.Report((double)totalRead / totalBytes); // 0.0 to 1.0
    }
}
```

---

## Summary

| Concept | Detail |
|---|---|
| What is FileStream | Concrete [[Stream]] subclass for byte-level file I/O |
| Constructor params | `path`, `FileMode`, `FileAccess`, `FileShare`, `bufferSize`, `useAsync` |
| FileMode | Controls open/create behavior: `Create`, `Open`, `OpenOrCreate`, `Append`, `Truncate`, `CreateNew` |
| FileAccess | `Read`, `Write`, `ReadWrite` |
| FileShare | Controls concurrent access: `None`, `Read`, `Write`, `ReadWrite`, `Delete` |
| Seeking | Full random access via `Seek(offset, SeekOrigin)` -- supports `Begin`, `Current`, `End` |
| Async I/O | Must set `useAsync: true` in constructor for true non-blocking I/O |
| Always dispose | Use `using` / `await using` to release file handles and flush buffers |
| Convenience shortcuts | `File.OpenRead()`, `File.OpenWrite()`, `File.Create()`, `File.Open()` |
