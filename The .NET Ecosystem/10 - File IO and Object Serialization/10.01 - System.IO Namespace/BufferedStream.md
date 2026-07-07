---
tags:
 - csharp
 - io
 - streams
---

## What Is BufferedStream?

`BufferedStream` is a [[Stream]] decorator that adds a temporary in-memory buffer between your code and another stream. Instead of hitting the underlying I/O (disk, network) on every small read or write, data accumulates in the buffer and is flushed in larger, more efficient chunks.

```
Without buffering:                     With BufferedStream:

Write(4 bytes)  →  disk                Write(4 bytes)  →  buffer
Write(4 bytes)  →  disk                Write(4 bytes)  →  buffer
Write(4 bytes)  →  disk                Write(4 bytes)  →  buffer
Write(4 bytes)  →  disk                Write(4 bytes)  →  buffer
                                       buffer full     →  disk (16 bytes at once)
4 disk operations                      1 disk operation
```

Each disk/network I/O call has fixed overhead regardless of size. Buffering amortizes that overhead across many small operations.


---

## When to Use BufferedStream

In most cases you **don't need it explicitly** because:
- `FileStream` already has an internal buffer (default 4096 bytes)
- `StreamReader`/`StreamWriter` also buffer internally

`BufferedStream` is useful when:
- You're wrapping a stream that has **no internal buffering** (e.g., `NetworkStream`)
- You want to **control the buffer size** independently of the inner stream
- You're wrapping a custom `Stream` implementation

```ad-note
Wrapping a `FileStream` in a `BufferedStream` is usually redundant — `FileStream` already buffers. You'd only do this if you want a buffer larger than `FileStream`'s or if you're layering streams (e.g., a `CryptoStream` around a `NetworkStream`).
```


---

## Creating a BufferedStream

```csharp
// Wrap any stream with a buffer (default buffer size: 4096 bytes)
using var networkStream = client.GetStream();
using var buffered = new BufferedStream(networkStream);

// Specify custom buffer size
using var bufferedLarge = new BufferedStream(networkStream, bufferSize: 65536);
```

The `BufferedStream` inherits from `Stream`, so it exposes the same `Read`/`Write`/`Seek`/`Flush` API — you can pass it anywhere a `Stream` is expected.


---

## Reading with BufferedStream

```csharp
using var raw = new NetworkStream(socket);
using var buffered = new BufferedStream(raw, bufferSize: 8192);
using var reader = new StreamReader(buffered);

// Each ReadLine may hit the buffer, not the network
string? line;
while ((line = reader.ReadLine()) != null)
{
    ProcessLine(line);
}
```

When `StreamReader` asks `BufferedStream` for data, `BufferedStream` reads a large chunk from the network into its internal buffer. Subsequent small reads are served from the buffer until it's depleted, then it refills.


---

## Writing with BufferedStream

```csharp
using var raw = new NetworkStream(socket);
using var buffered = new BufferedStream(raw, bufferSize: 8192);
using var writer = new StreamWriter(buffered);

for (int i = 0; i < 10000; i++)
{
    writer.WriteLine($"Line {i}");  // writes go to the buffer
}
// Dispose flushes the buffer → flushes to network
```


---

## Flushing

`Flush()` pushes any buffered data to the underlying stream and then flushes the underlying stream too:

```csharp
buffered.Flush();  // buffer → inner stream → underlying I/O
```

`Dispose()` automatically flushes before closing.

```ad-warning
If you don't call `Flush()` or `Dispose()`, buffered data may be lost. Always use `using` to ensure disposal.
```


---

## Buffer Size Guidance

| Scenario | Suggested buffer size |
|---|---|
| Network I/O (typical) | 4096 – 8192 bytes |
| High-throughput network | 65536+ bytes |
| Wrapping a custom stream | 4096 (default) is usually fine |
| File I/O | Usually unnecessary — `FileStream` already buffers |

Larger buffers reduce I/O calls but use more memory. The default of 4096 is a good balance for most scenarios.


---

## BufferedStream vs Other Buffering

| Source of buffering | Built-in? | Buffer size |
|---|---|---|
| `FileStream` | Yes | Constructor parameter (default 4096) |
| `StreamReader` / `StreamWriter` | Yes | Internal (default 1024 chars) |
| `BufferedStream` | Explicit wrapper | Constructor parameter (default 4096) |
| `NetworkStream` | **No** | Must wrap with `BufferedStream` |


---

## See Also

- [[Stream]] — the abstract base class
- [[FileStream]] — already has internal buffering
- [[MemoryStream]] — in-memory stream (no I/O overhead, so buffering is irrelevant)
- [[StreamReader and StreamWriter]] — text layer, also internally buffered
