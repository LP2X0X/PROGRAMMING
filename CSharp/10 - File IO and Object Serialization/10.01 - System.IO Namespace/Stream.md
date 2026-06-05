---
tags:
 - csharp
 - io
 - streams
---

## Stream -- The Abstract Base Class for All Byte I/O

A **stream** is an abstraction that represents a sequence of bytes you can read from, write to, or both. The fundamental insight is that ==you do not care where the bytes come from or where they go==. The same `Stream` API works whether the bytes live in a file on disk, in memory, on a network socket, behind a compression algorithm, or inside an encryption layer.

The `System.IO.Stream` class is **abstract** -- you never instantiate `Stream` directly. Instead, you work with concrete subclasses like [[FileStream]], [[MemoryStream]], `BufferedStream`, `NetworkStream`, `GZipStream`, `CryptoStream`, and others. Every one of them inherits from `Stream` and exposes the same core methods for reading, writing, seeking, and disposing.

This design means any code that accepts a `Stream` parameter works with **any** backing store. You can write a method that processes a `FileStream` today and swap in a `MemoryStream` for unit tests tomorrow -- zero code changes.

---

## The Stream Hierarchy

```
Stream (abstract)
├── FileStream            -- reads/writes bytes to a file on disk
├── MemoryStream          -- reads/writes bytes to an in-memory byte array
├── BufferedStream        -- wraps another stream, adds a buffer for fewer I/O calls
├── NetworkStream         -- reads/writes bytes over a TCP socket (System.Net.Sockets)
├── GZipStream            -- compresses/decompresses via gzip (System.IO.Compression)
├── DeflateStream         -- compresses/decompresses via deflate
├── CryptoStream          -- encrypts/decrypts via a cryptographic transform (System.Security.Cryptography)
├── PipeStream            -- inter-process communication via named/anonymous pipes (System.IO.Pipes)
│   ├── NamedPipeClientStream
│   ├── NamedPipeServerStream
│   └── AnonymousPipeServerStream / AnonymousPipeClientStream
└── UnmanagedMemoryStream -- direct access to unmanaged memory blocks
```

Readers and writers sit **on top of** streams as adapters:

```
┌─────────────────────────────────────────────────────────────┐
│  StreamReader / StreamWriter        (text layer)            │
│  BinaryReader / BinaryWriter        (primitive type layer)  │
├─────────────────────────────────────────────────────────────┤
│  Stream (FileStream, MemoryStream, NetworkStream, etc.)     │
│  (byte layer -- reads/writes raw byte[])                    │
└─────────────────────────────────────────────────────────────┘
```

See [[StreamReader and StreamWriter]] and [[BinaryReader and BinaryWriter]] for the higher-level adapters.

---

## The Decorator Pattern -- Streams Wrapping Streams

One of the most powerful aspects of the stream architecture is that **streams can wrap other streams**. This is the classic **decorator pattern** -- each wrapper adds behavior (buffering, compression, encryption) without changing the API.

```csharp
// Layer 1: FileStream reads/writes raw bytes on disk
using FileStream fileStream = new FileStream("archive.gz", FileMode.Create);

// Layer 2: GZipStream compresses bytes before passing to FileStream
using GZipStream gzipStream = new GZipStream(fileStream, CompressionLevel.Optimal);

// Layer 3: StreamWriter converts text to bytes before passing to GZipStream
using StreamWriter writer = new StreamWriter(gzipStream);

// From the caller's perspective, you're just writing text
writer.WriteLine("This text will be compressed and written to a .gz file");
```

The data flows through the pipeline:

```
writer.WriteLine("text")
  → StreamWriter encodes text to UTF-8 bytes
    → GZipStream compresses those bytes
      → FileStream writes compressed bytes to disk
```

When reading, the flow reverses:

```
reader.ReadLine()
  ← StreamWriter decodes UTF-8 bytes to text
    ← GZipStream decompresses bytes
      ← FileStream reads compressed bytes from disk
```

You can stack as many layers as needed. A common real-world example:

```csharp
// Encrypted, compressed file I/O
FileStream → BufferedStream → CryptoStream → GZipStream → StreamWriter
```

---

## Key Properties

Every `Stream` exposes these properties to describe its capabilities and current state.

### Capability Properties

| Property | Type | Description |
|---|---|---|
| `CanRead` | `bool` | `true` if the stream supports reading. A write-only stream returns `false`. |
| `CanWrite` | `bool` | `true` if the stream supports writing. A read-only stream returns `false`. |
| `CanSeek` | `bool` | `true` if the stream supports seeking (random access). Network and compression streams typically return `false`. |
| `CanTimeout` | `bool` | `true` if the stream supports read/write timeouts. Mostly relevant for `NetworkStream`. |

```ad-note
title: Always Check Capabilities
Before calling `Seek()`, `Length`, or `Position` (set), check `CanSeek`. Calling these on a non-seekable stream throws `NotSupportedException`. The same applies to `Read`/`Write` -- check `CanRead`/`CanWrite` if the stream's capabilities are unknown at compile time.
```

### State Properties

| Property | Type | Description |
|---|---|---|
| `Length` | `long` | Total number of bytes in the stream. Not all streams support this -- `NetworkStream` and `GZipStream` throw `NotSupportedException`. |
| `Position` | `long` | The current read/write cursor position (0-based byte offset from the beginning). You can get or set this on seekable streams. Setting `Position` is equivalent to calling `Seek(value, SeekOrigin.Begin)`. |
| `ReadTimeout` | `int` | Timeout in milliseconds for read operations. Only meaningful when `CanTimeout` is `true`. |
| `WriteTimeout` | `int` | Timeout in milliseconds for write operations. Only meaningful when `CanTimeout` is `true`. |

```csharp
using var fs = new FileStream("data.bin", FileMode.Open, FileAccess.Read);

Console.WriteLine(fs.CanRead);   // True
Console.WriteLine(fs.CanWrite);  // False (opened with FileAccess.Read)
Console.WriteLine(fs.CanSeek);   // True (file streams support seeking)
Console.WriteLine(fs.Length);    // total file size in bytes
Console.WriteLine(fs.Position);  // 0 (haven't read anything yet)
```

---

## Core Methods -- Synchronous

### Read(byte[], int, int)

Reads a sequence of bytes from the stream into a buffer.

```csharp
int Read(byte[] buffer, int offset, int count)
```

- **buffer** -- the byte array to fill
- **offset** -- where in the buffer to start writing (usually 0)
- **count** -- maximum number of bytes to read
- **Returns** -- the actual number of bytes read (0 means end-of-stream)

```ad-warning
title: Read May Return Fewer Bytes Than Requested
`Read()` can return **fewer** bytes than `count` even when there are more bytes available. This is especially common with `NetworkStream`, where data arrives in packets. ==Always use a loop to read until you get all the data you need.==
```

```csharp
using var fs = new FileStream("data.bin", FileMode.Open, FileAccess.Read);
byte[] buffer = new byte[4096];
int bytesRead;

// Correct: loop until Read returns 0 (end-of-stream)
while ((bytesRead = fs.Read(buffer, 0, buffer.Length)) > 0)
{
    // Process buffer[0..bytesRead-1]
    // Note: only the first 'bytesRead' bytes are valid
    ProcessChunk(buffer, bytesRead);
}
```

### Read(Span\<byte\>) -- Modern Overload (.NET Core 2.1+)

```csharp
int Read(Span<byte> buffer)
```

The `Span<byte>` overload avoids the offset/count parameters and works with stack-allocated buffers:

```csharp
Span<byte> buffer = stackalloc byte[256];
int bytesRead = stream.Read(buffer);
```

### ReadByte()

Reads a single byte from the stream.

```csharp
int ReadByte()
```

- Returns the byte as an `int` (0-255), or **-1** if the end of the stream is reached.

```csharp
int b;
while ((b = stream.ReadByte()) != -1)
{
    byte value = (byte)b;
    // process single byte
}
```

### ReadExactly(byte[], int, int) -- .NET 7+

Unlike `Read()`, this method guarantees it reads **exactly** the requested number of bytes, or throws `EndOfStreamException` if the stream ends early.

```csharp
byte[] header = new byte[4];
stream.ReadExactly(header, 0, 4); // throws if fewer than 4 bytes remain
```

### Write(byte[], int, int)

Writes a sequence of bytes to the stream.

```csharp
void Write(byte[] buffer, int offset, int count)
```

- **buffer** -- the bytes to write
- **offset** -- where in the buffer to start reading from
- **count** -- number of bytes to write

```csharp
byte[] data = Encoding.UTF8.GetBytes("Hello, Stream!");
stream.Write(data, 0, data.Length);
```

### Write(ReadOnlySpan\<byte\>) -- Modern Overload

```csharp
stream.Write(new byte[] { 0x48, 0x65, 0x6C, 0x6C, 0x6F });
```

### WriteByte(byte)

Writes a single byte to the stream.

```csharp
stream.WriteByte(0xFF);
```

### Seek(long, SeekOrigin)

Moves the read/write position to a specific location. Only works on seekable streams (`CanSeek == true`).

```csharp
long Seek(long offset, SeekOrigin origin)
```

**SeekOrigin values:**

| Value | Meaning | Example |
|---|---|---|
| `SeekOrigin.Begin` | Offset from the start of the stream | `Seek(0, SeekOrigin.Begin)` -- go to byte 0 |
| `SeekOrigin.Current` | Offset from the current position | `Seek(-4, SeekOrigin.Current)` -- back 4 bytes |
| `SeekOrigin.End` | Offset from the end of the stream | `Seek(-10, SeekOrigin.End)` -- 10 bytes before end |

```csharp
using var fs = new FileStream("data.bin", FileMode.Open);

fs.Seek(0, SeekOrigin.Begin);    // go to the very start
fs.Seek(100, SeekOrigin.Begin);  // go to byte 100
fs.Seek(-10, SeekOrigin.End);    // go to 10 bytes before end
fs.Seek(5, SeekOrigin.Current);  // advance 5 bytes from current position
```

### Flush()

Forces any buffered data to be written to the underlying store. Many streams buffer writes for performance -- `Flush()` ensures everything is actually committed.

```csharp
stream.Write(data, 0, data.Length);
stream.Flush(); // ensure bytes are written to disk / network / etc.
```

```ad-note
title: Flush vs Close
`Close()` and `Dispose()` both call `Flush()` internally before releasing resources. You typically only need to call `Flush()` manually when you want to ensure intermediate data is committed without closing the stream (e.g., logging to a file that stays open).
```

### CopyTo(Stream)

Copies the entire contents of the current stream to a destination stream. Reads from the current position to the end.

```csharp
void CopyTo(Stream destination)
void CopyTo(Stream destination, int bufferSize)
```

```csharp
using var source = File.OpenRead("input.dat");
using var dest = File.Create("output.dat");
source.CopyTo(dest); // copies everything from source to dest
```

### Close() / Dispose()

Releases all resources held by the stream (file handles, network sockets, memory, etc.).

- `Close()` calls `Dispose()` internally -- they are equivalent.
- **Always** use `using` statements instead of calling `Close()` or `Dispose()` manually.

```csharp
// Preferred: using declaration (C# 8+)
using var fs = new FileStream("data.bin", FileMode.Open);
// fs is automatically disposed when it goes out of scope

// Alternative: using block
using (var fs2 = new FileStream("data.bin", FileMode.Open))
{
    // work with fs2
} // disposed here
```

---

## Core Methods -- Asynchronous

Every synchronous I/O method on `Stream` has an `async` counterpart. For I/O-bound work (files, network), the async versions are strongly preferred because they free up threads while waiting for the I/O to complete.

| Synchronous | Asynchronous | Notes |
|---|---|---|
| `Read(byte[], int, int)` | `ReadAsync(byte[], int, int, CancellationToken)` | Returns `Task<int>` |
| `ReadByte()` | *(no direct async equivalent)* | Use `ReadAsync` with a 1-byte buffer |
| `Write(byte[], int, int)` | `WriteAsync(byte[], int, int, CancellationToken)` | Returns `Task` |
| `WriteByte(byte)` | *(no direct async equivalent)* | Use `WriteAsync` with a 1-byte buffer |
| `CopyTo(Stream)` | `CopyToAsync(Stream, CancellationToken)` | Returns `Task` |
| `Flush()` | `FlushAsync(CancellationToken)` | Returns `Task` |

```csharp
using var fs = new FileStream("data.bin", FileMode.Open, FileAccess.Read,
    FileShare.Read, bufferSize: 4096, useAsync: true);

byte[] buffer = new byte[4096];
int bytesRead;

while ((bytesRead = await fs.ReadAsync(buffer, 0, buffer.Length)) > 0)
{
    await ProcessChunkAsync(buffer, bytesRead);
}
```

```ad-important
title: True Async Requires useAsync on FileStream
For `FileStream` specifically, you must pass `useAsync: true` in the constructor for truly asynchronous I/O. Without it, `ReadAsync` and `WriteAsync` simply wrap the synchronous calls on a thread pool thread, which is **not** true async I/O -- it just hides the blocking. See [[FileStream]] for details.
```

### Memory\<byte\> Overloads (.NET Core 2.1+)

Modern .NET provides `Memory<byte>` and `ReadOnlyMemory<byte>` overloads that are more efficient and `Span`-friendly:

```csharp
Memory<byte> buffer = new byte[4096];
int bytesRead = await stream.ReadAsync(buffer);

ReadOnlyMemory<byte> data = new byte[] { 1, 2, 3 };
await stream.WriteAsync(data);
```

---

## IDisposable -- Always Use `using`

`Stream` implements `IDisposable`. Streams hold onto **unmanaged resources** -- file handles, socket descriptors, memory mappings. If you forget to dispose, those resources leak.

```ad-warning
title: Disposing Writers Flushes Data
When you dispose a `StreamWriter` or `BinaryWriter`, it flushes any buffered data to the underlying stream, then disposes the stream. If you forget to dispose the writer, ==buffered data may never reach the file==, causing silent data loss.
```

```csharp
// CORRECT: using declaration ensures disposal
using var stream = new FileStream("data.bin", FileMode.Create);
using var writer = new BinaryWriter(stream);
writer.Write(42);
// writer and stream are disposed at end of scope -- data is flushed

// WRONG: no using -- if an exception occurs, the stream leaks
var stream2 = new FileStream("data.bin", FileMode.Create);
stream2.Write(new byte[] { 1, 2, 3 }, 0, 3);
stream2.Close(); // might never execute if an exception is thrown above
```

---

## Practical Example -- Buffered File Copy

This example demonstrates reading a file in chunks, which is how you handle large files without loading them entirely into memory:

```csharp
public static void CopyFile(string sourcePath, string destPath, int bufferSize = 81920)
{
    using var source = new FileStream(sourcePath, FileMode.Open, FileAccess.Read);
    using var dest = new FileStream(destPath, FileMode.Create, FileAccess.Write);

    byte[] buffer = new byte[bufferSize];
    int bytesRead;

    while ((bytesRead = source.Read(buffer, 0, buffer.Length)) > 0)
    {
        dest.Write(buffer, 0, bytesRead); // write only the bytes we actually read
    }
}
```

Or more concisely using `CopyTo`:

```csharp
public static void CopyFile(string sourcePath, string destPath)
{
    using var source = File.OpenRead(sourcePath);
    using var dest = File.Create(destPath);
    source.CopyTo(dest);
}
```

Async version:

```csharp
public static async Task CopyFileAsync(string sourcePath, string destPath,
    CancellationToken ct = default)
{
    await using var source = new FileStream(sourcePath, FileMode.Open, FileAccess.Read,
        FileShare.Read, bufferSize: 4096, useAsync: true);
    await using var dest = new FileStream(destPath, FileMode.Create, FileAccess.Write,
        FileShare.None, bufferSize: 4096, useAsync: true);

    await source.CopyToAsync(dest, ct);
}
```

```ad-note
title: await using
In C# 8+, `await using` is the async equivalent of `using`. It calls `DisposeAsync()` instead of `Dispose()`, allowing the cleanup itself to be async (e.g., flushing a network stream).
```

---

## Practical Example -- Seeking and Random Access

Seeking is useful when you need to read or modify specific parts of a file without processing it sequentially:

```csharp
using var fs = new FileStream("records.bin", FileMode.Open, FileAccess.ReadWrite);

// Each record is 100 bytes. Jump to record #5 (0-indexed)
int recordSize = 100;
int recordIndex = 5;
fs.Seek(recordIndex * recordSize, SeekOrigin.Begin);

// Read record #5
byte[] record = new byte[recordSize];
fs.ReadExactly(record, 0, recordSize);

// Modify and write back to the same position
fs.Seek(recordIndex * recordSize, SeekOrigin.Begin);
record[0] = 0xFF; // modify first byte
fs.Write(record, 0, record.Length);
```

---

## Stream.Null -- The /dev/null of Streams

`Stream.Null` is a singleton stream that discards all writes and returns zero bytes on reads. Useful as a no-op placeholder:

```csharp
// Discard output -- useful in testing or when a Stream parameter is required but output is unwanted
Stream.Null.Write(data, 0, data.Length); // silently discarded
int read = Stream.Null.Read(buffer, 0, buffer.Length); // always returns 0
```

---

## Summary

| Concept | Detail |
|---|---|
| What is a Stream | Abstract base class representing a sequence of bytes from any source |
| Key subclasses | [[FileStream]], [[MemoryStream]], `BufferedStream`, `NetworkStream`, `GZipStream`, `CryptoStream` |
| Capability checks | `CanRead`, `CanWrite`, `CanSeek` -- always check before using features |
| Core read/write | `Read(byte[], offset, count)`, `Write(byte[], offset, count)`, `ReadByte()`, `WriteByte()` |
| Seeking | `Seek(offset, SeekOrigin)` -- random access on seekable streams |
| Resource management | `IDisposable` -- always use `using` to avoid leaks and data loss |
| Decorator pattern | Streams wrap other streams to add behavior (buffering, compression, encryption) |
| Async I/O | `ReadAsync`, `WriteAsync`, `CopyToAsync`, `FlushAsync` -- prefer for I/O-bound work |
| Readers/Writers | [[StreamReader and StreamWriter]] for text, [[BinaryReader and BinaryWriter]] for primitives -- layered on top of `Stream` |
