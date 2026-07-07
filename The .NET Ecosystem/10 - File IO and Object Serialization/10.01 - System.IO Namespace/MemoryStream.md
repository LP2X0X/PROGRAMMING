---
tags:
 - csharp
 - io
 - streams
---

## MemoryStream -- A Stream Backed by an In-Memory Byte Array

`MemoryStream` is a [[Stream]] subclass that stores its data in a **byte array in memory** rather than on disk or over a network. It behaves exactly like any other stream -- you can read, write, seek, and get its length -- but all data lives in RAM and vanishes when the `MemoryStream` is garbage collected.

### Why MemoryStream Exists

`MemoryStream` fills a critical gap in the stream architecture. Many APIs accept or return `Stream` -- they don't care where the bytes come from. `MemoryStream` lets you participate in these APIs without touching the file system:

- **Building binary data in memory** before writing to disk, sending over the network, or returning from an API
- **Converting between `Stream` and `byte[]`** -- many APIs accept `Stream`, but you have a `byte[]` (or vice versa)
- **Unit testing** -- mock a file by passing a `MemoryStream` to code that expects a `Stream` parameter
- **Buffering** -- accumulate data from multiple writes, then extract the final `byte[]`
- **Serialization / deserialization pipelines** -- serialize to a `MemoryStream`, then send the bytes wherever needed
- **Stream adapters** -- wrap with [[StreamReader and StreamWriter]] or [[BinaryReader and BinaryWriter]] for text/binary operations entirely in memory

---

## Creating a MemoryStream

### Empty (Writable, Growable)

Creates a `MemoryStream` with an empty, resizable internal buffer. Start writing immediately -- the buffer grows automatically.

```csharp
var ms = new MemoryStream();
// Internal buffer starts empty and grows as you write
```

### With Initial Capacity

Pre-allocates the internal buffer to avoid repeated resizing when you know the approximate size.

```csharp
var ms = new MemoryStream(capacity: 1024); // pre-allocate 1 KB
// Still empty (Length == 0), but the internal buffer is 1024 bytes
// Avoids resizing if you write <= 1024 bytes
```

### From an Existing byte[]

Wraps an existing byte array. The `MemoryStream` reads from (and optionally writes to) this array directly -- no copy is made.

```csharp
byte[] data = { 1, 2, 3, 4, 5 };

// Read-only by default (writable: false)
var ms1 = new MemoryStream(data);
// ms1.Length == 5, ms1.Position == 0, ms1.CanWrite == true (confusingly)
// But the buffer is NOT resizable -- Write beyond Length throws

// Writable, specific region
var ms2 = new MemoryStream(data, index: 0, count: data.Length, writable: true);

// Read-only, specific region, publiclyVisible controls GetBuffer()
var ms3 = new MemoryStream(data, index: 1, count: 3, writable: false, publiclyVisible: true);
// ms3 contains bytes { 2, 3, 4 }
```

```ad-warning
title: MemoryStream from byte[] Is NOT Resizable
When you create a `MemoryStream` from an existing `byte[]`, the internal buffer is **fixed to that array**. You can write within the existing bounds, but you cannot grow the stream beyond the array's length. Attempting to write past the end throws `NotSupportedException`. Only a `MemoryStream` created without a source array (or with just a capacity) is resizable.
```

### Constructor Summary

| Constructor | Resizable | Writable | Backed By |
|---|---|---|---|
| `new MemoryStream()` | Yes | Yes | New internal array (grows automatically) |
| `new MemoryStream(capacity)` | Yes | Yes | New internal array, pre-allocated |
| `new MemoryStream(byte[])` | No | Yes (within bounds) | The provided array |
| `new MemoryStream(byte[], writable)` | No | `writable` param | The provided array |
| `new MemoryStream(byte[], index, count)` | No | Yes (within bounds) | Slice of the provided array |
| `new MemoryStream(byte[], index, count, writable, publiclyVisible)` | No | `writable` param | Slice of the provided array |

---

## Writing and Reading Back

The most common pattern is: write data, rewind to the beginning, then read it back.

### With StreamWriter / StreamReader

```csharp
using var ms = new MemoryStream();

// Write text into the MemoryStream
using (var writer = new StreamWriter(ms, Encoding.UTF8, bufferSize: 1024, leaveOpen: true))
{
    writer.WriteLine("Line 1");
    writer.WriteLine("Line 2");
    writer.WriteLine("Line 3");
    writer.Flush(); // ensure all buffered text is written to the MemoryStream
}

// Reset position to the beginning before reading
ms.Position = 0;

// Read text back from the MemoryStream
using var reader = new StreamReader(ms);
string content = reader.ReadToEnd();
Console.WriteLine(content);
// Output:
// Line 1
// Line 2
// Line 3
```

```ad-important
title: Always Reset Position Before Reading
After writing, the stream's `Position` is at the **end** of the written data. If you try to read without resetting `Position` to 0, you'll read zero bytes because there's nothing after the current position. ==Always set `ms.Position = 0` (or call `ms.Seek(0, SeekOrigin.Begin)`) before reading.==
```

### With BinaryWriter / BinaryReader

```csharp
using var ms = new MemoryStream();

// Write binary data
using (var writer = new BinaryWriter(ms, Encoding.UTF8, leaveOpen: true))
{
    writer.Write(42);          // 4 bytes
    writer.Write(3.14);        // 8 bytes
    writer.Write("Hello");     // 6 bytes (1 length + 5 chars)
}

// Rewind
ms.Position = 0;

// Read binary data back
using var reader = new BinaryReader(ms, Encoding.UTF8);
int number = reader.ReadInt32();     // 42
double pi = reader.ReadDouble();     // 3.14
string text = reader.ReadString();   // "Hello"
```

### Direct Byte Writing / Reading

```csharp
using var ms = new MemoryStream();

// Write raw bytes
byte[] data = { 0xDE, 0xAD, 0xBE, 0xEF };
ms.Write(data, 0, data.Length);
ms.WriteByte(0xFF);

// Rewind and read
ms.Position = 0;
byte[] buffer = new byte[5];
int bytesRead = ms.Read(buffer, 0, buffer.Length);
// buffer: { 0xDE, 0xAD, 0xBE, 0xEF, 0xFF }, bytesRead: 5
```

---

## The leaveOpen Parameter

When you wrap a `MemoryStream` with a `StreamWriter`, `StreamReader`, `BinaryWriter`, or `BinaryReader`, disposing the wrapper **also disposes the MemoryStream** by default. If you need the `MemoryStream` to survive, set `leaveOpen: true`.

```csharp
using var ms = new MemoryStream();

// WITHOUT leaveOpen: disposing the writer kills the MemoryStream
using (var writer = new StreamWriter(ms))
{
    writer.Write("data");
}
// ms is now disposed -- ms.ToArray() throws ObjectDisposedException!

// WITH leaveOpen: MemoryStream survives
using var ms2 = new MemoryStream();
using (var writer = new StreamWriter(ms2, Encoding.UTF8, 1024, leaveOpen: true))
{
    writer.Write("data");
}
ms2.Position = 0; // OK -- ms2 is still alive
byte[] bytes = ms2.ToArray(); // OK
```

```ad-important
title: leaveOpen Is Essential for Write-Then-Read Patterns
Almost every time you use a `MemoryStream` with a reader or writer, you need `leaveOpen: true` so you can continue using the stream after the writer is disposed. This is the most common `MemoryStream` mistake.
```

---

## Getting the Underlying Bytes

### ToArray()

Creates a **new byte array** containing a copy of the stream's data, sized exactly to `Length` (not the internal buffer capacity).

```csharp
using var ms = new MemoryStream();
ms.Write(new byte[] { 1, 2, 3, 4, 5 }, 0, 5);

byte[] result = ms.ToArray();
// result: { 1, 2, 3, 4, 5 } -- length 5
// This is a COPY -- modifying result does not affect ms
```

### GetBuffer()

Returns a **direct reference** to the internal byte array. This array may be **larger** than the actual data -- use `ms.Length` to know how many bytes are valid.

```csharp
using var ms = new MemoryStream();
ms.Write(new byte[] { 1, 2, 3, 4, 5 }, 0, 5);

byte[] buffer = ms.GetBuffer();
// buffer.Length might be 256 (internal capacity), but only buffer[0..4] is valid data
// ms.Length == 5 -- only the first 5 bytes are meaningful

// WARNING: buffer may have garbage bytes after index 4
```

```ad-warning
title: GetBuffer() Restrictions
`GetBuffer()` throws `UnauthorizedAccessException` if the `MemoryStream` was created from a `byte[]` with `publiclyVisible: false` (the default for the `byte[]` constructors). Only `MemoryStream` created without a source array, or with `publiclyVisible: true`, allows `GetBuffer()`.

Use `TryGetBuffer()` for a safe alternative that doesn't throw.
```

### TryGetBuffer(out ArraySegment\<byte\>) -- Safe Alternative

```csharp
using var ms = new MemoryStream();
ms.Write(new byte[] { 1, 2, 3 }, 0, 3);

if (ms.TryGetBuffer(out ArraySegment<byte> segment))
{
    // segment.Array is the internal buffer
    // segment.Offset is the start index (usually 0)
    // segment.Count is the valid byte count (== ms.Length)
    byte[] array = segment.Array!;
    int validBytes = segment.Count; // 3
}
```

### Comparison Table

| Method | Returns | Allocation | Safe | Notes |
|---|---|---|---|---|
| `ToArray()` | `byte[]` (exact size) | Copies data into new array | Always works | Use when you need an independent copy |
| `GetBuffer()` | `byte[]` (internal, may be larger) | No allocation -- direct reference | May throw | Use when performance matters and you track `Length` |
| `TryGetBuffer()` | `ArraySegment<byte>` | No allocation | Always works (returns false if inaccessible) | Best of both -- safe and zero-copy |

---

## MemoryStream vs FileStream

| Aspect | MemoryStream | [[FileStream]] |
|---|---|---|
| **Backed by** | In-memory byte array (RAM) | File on disk |
| **Speed** | Very fast (memory speed) | Slower (disk I/O latency) |
| **Size limit** | Available process memory (practical limit ~1-2 GB on 32-bit, much more on 64-bit) | Disk space |
| **Persistence** | Lost when object is collected / application exits | Persists on disk across process restarts |
| **Seeking** | `CanSeek == true` | `CanSeek == true` |
| **Use when** | Temporary data, testing, buffering, in-memory transformation | Permanent storage, large files, data that must survive process exit |

---

## Common Use Cases with Examples

### Use Case 1: Unit Testing Code That Takes a Stream

```csharp
// Production code accepts a Stream -- doesn't know or care where bytes come from
public static int CountLines(Stream stream)
{
    using var reader = new StreamReader(stream, leaveOpen: true);
    int count = 0;
    while (reader.ReadLine() != null)
        count++;
    return count;
}

// Unit test -- no file system needed
[Fact]
public void CountLines_ReturnsCorrectCount()
{
    string testData = "Line 1\nLine 2\nLine 3\n";
    using var ms = new MemoryStream(Encoding.UTF8.GetBytes(testData));

    int result = CountLines(ms);

    Assert.Equal(3, result);
}
```

### Use Case 2: Building Binary Data for Network Transmission

```csharp
public static byte[] BuildPacket(int commandId, string payload)
{
    using var ms = new MemoryStream();
    using var writer = new BinaryWriter(ms, Encoding.UTF8, leaveOpen: true);

    writer.Write((byte)0xAA);           // start marker
    writer.Write(commandId);            // 4-byte command ID
    writer.Write(payload);              // length-prefixed string
    writer.Write((byte)0x55);           // end marker
    writer.Flush();

    return ms.ToArray(); // get the final byte[] to send over the network
}
```

### Use Case 3: Converting a Stream to byte[] and Back

```csharp
// Stream → byte[]
public static byte[] StreamToByteArray(Stream source)
{
    // If the source is already a MemoryStream, take a shortcut
    if (source is MemoryStream sourceMs)
        return sourceMs.ToArray();

    using var ms = new MemoryStream();
    source.CopyTo(ms);
    return ms.ToArray();
}

// byte[] → Stream
public static MemoryStream ByteArrayToStream(byte[] data)
{
    return new MemoryStream(data, writable: false);
}
```

### Use Case 4: In-Memory Compression

```csharp
using System.IO.Compression;

public static byte[] Compress(byte[] data)
{
    using var output = new MemoryStream();
    using (var gzip = new GZipStream(output, CompressionLevel.Optimal, leaveOpen: true))
    {
        gzip.Write(data, 0, data.Length);
    } // GZipStream must be disposed/closed to finalize the compression

    return output.ToArray();
}

public static byte[] Decompress(byte[] compressedData)
{
    using var input = new MemoryStream(compressedData);
    using var gzip = new GZipStream(input, CompressionMode.Decompress);
    using var output = new MemoryStream();

    gzip.CopyTo(output);
    return output.ToArray();
}
```

### Use Case 5: Serialization Pipeline

```csharp
using System.Text.Json;

public static byte[] SerializeToBytes<T>(T obj)
{
    using var ms = new MemoryStream();
    JsonSerializer.Serialize(ms, obj);
    return ms.ToArray();
}

public static T DeserializeFromBytes<T>(byte[] bytes)
{
    using var ms = new MemoryStream(bytes);
    return JsonSerializer.Deserialize<T>(ms)!;
}
```

---

## Capacity vs Length vs Position

Understanding these three properties is key to avoiding confusion with `MemoryStream`.

| Property | Meaning |
|---|---|
| `Capacity` | The total size of the internal buffer in bytes. May be larger than `Length`. Grows automatically (doubles) when writes exceed it. |
| `Length` | The number of **valid** bytes in the stream. Bytes 0 through `Length - 1` contain meaningful data. |
| `Position` | The current read/write cursor (0-based offset). Advances as you read or write. |

```csharp
var ms = new MemoryStream(capacity: 16);

Console.WriteLine(ms.Capacity);  // 16 (pre-allocated buffer)
Console.WriteLine(ms.Length);    // 0  (no data written yet)
Console.WriteLine(ms.Position);  // 0  (cursor at the start)

ms.Write(new byte[] { 1, 2, 3 }, 0, 3);

Console.WriteLine(ms.Capacity);  // 16 (buffer hasn't needed to grow)
Console.WriteLine(ms.Length);    // 3  (3 bytes written)
Console.WriteLine(ms.Position);  // 3  (cursor moved forward by 3)

ms.Position = 0;                  // rewind for reading
Console.WriteLine(ms.Position);  // 0  (back at the start)
Console.WriteLine(ms.Length);    // 3  (unchanged -- Length tracks data, not cursor)
```

```ad-note
title: SetLength() Can Truncate or Extend
You can call `ms.SetLength(newLength)` to explicitly resize the valid data region. Setting it smaller truncates data. Setting it larger pads with zero bytes. This only works on resizable `MemoryStream` instances.
```

---

## Dispose Behavior

Disposing a `MemoryStream` marks it as unusable (reads/writes throw `ObjectDisposedException`), but it does **not** release the underlying byte array immediately -- the GC handles that. However, ==you should still always dispose== because:

1. **Consistency** -- all streams should be disposed via `using`
2. **Future-proofing** -- your code might be refactored to use a different stream type that does hold unmanaged resources
3. **`ToArray()` works after Dispose** -- this is a quirk; `ToArray()` still succeeds on a disposed `MemoryStream`, but `Read()`/`Write()` throw

```csharp
var ms = new MemoryStream();
ms.Write(new byte[] { 1, 2, 3 }, 0, 3);
ms.Dispose();

byte[] data = ms.ToArray();  // Still works! Returns { 1, 2, 3 }
ms.Write(new byte[] { 4 }, 0, 1); // Throws ObjectDisposedException
```

---

## Summary

| Concept | Detail |
|---|---|
| What it is | A [[Stream]] backed by an in-memory byte array instead of a file |
| Primary uses | Temporary data buffering, unit testing, byte[] conversion, in-memory serialization |
| Creating | `new MemoryStream()` (growable), `new MemoryStream(byte[])` (fixed to array), `new MemoryStream(capacity)` (pre-allocated) |
| Position trap | ==Reset `Position` to 0 before reading after writes== |
| Getting bytes | `ToArray()` (safe copy), `GetBuffer()` (direct reference, may be larger), `TryGetBuffer()` (safe, zero-copy) |
| leaveOpen | Set `true` on wrappers (`StreamWriter`, `BinaryWriter`) to keep the `MemoryStream` alive after wrapper disposal |
| Resizability | Only growable when created without a source `byte[]`. Wrapping an existing array is fixed-size. |
| Persistence | None -- data is lost when the object is collected. Use [[FileStream]] for persistence. |
| Speed | RAM-speed. Orders of magnitude faster than disk I/O. |
