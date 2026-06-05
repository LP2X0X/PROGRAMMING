---
tags:
 - csharp
 - io
 - text-io
---

## StreamReader and StreamWriter -- Text I/O Over Streams

`StreamReader` and `StreamWriter` are the **text-level adapters** that sit on top of a [[Stream]]. While a `Stream` deals in raw bytes, `StreamReader`/`StreamWriter` deal in **characters and strings**. They handle the critical job of **encoding** -- converting between the bytes in the stream and the `char`/`string` types in .NET.

```
┌──────────────────────────────────────────────────┐
│  Your Code                                       │
│  reader.ReadLine()  /  writer.WriteLine("text")  │
├──────────────────────────────────────────────────┤
│  StreamReader / StreamWriter                     │
│  (converts between chars/strings and bytes)      │
│  (handles encoding: UTF-8, ASCII, Unicode, etc.) │
├──────────────────────────────────────────────────┤
│  Stream (FileStream, MemoryStream, etc.)         │
│  (raw byte[] I/O)                                │
└──────────────────────────────────────────────────┘
```

Both classes inherit from the abstract `TextReader` / `TextWriter` base classes respectively (not from `Stream`). They are **not** streams themselves -- they are adapters that **wrap** a stream.

---

## StreamReader -- Reading Text

### Creating a StreamReader

There are multiple ways to create a `StreamReader`, depending on whether you already have a `Stream` or just a file path.

```csharp
// From a file path (creates an internal FileStream automatically)
using var reader1 = new StreamReader("log.txt");

// From a file path with explicit encoding
using var reader2 = new StreamReader("log.txt", Encoding.UTF8);

// From an existing stream
using var fs = new FileStream("log.txt", FileMode.Open, FileAccess.Read);
using var reader3 = new StreamReader(fs);

// From an existing stream with encoding and leaveOpen
using var reader4 = new StreamReader(fs, Encoding.UTF8, detectEncodingFromByteOrderMarks: true,
    bufferSize: 1024, leaveOpen: true);

// Via File convenience method
using StreamReader reader5 = File.OpenText("log.txt");
```

### Key Constructor Parameters

| Parameter | Default | Description |
|---|---|---|
| `encoding` | UTF-8 (no BOM) | The text encoding to use when converting bytes to chars |
| `detectEncodingFromByteOrderMarks` | `true` | If `true`, examines the first bytes (BOM) to auto-detect encoding |
| `bufferSize` | 1024 | Internal character buffer size |
| `leaveOpen` | `false` | If `true`, disposing the `StreamReader` does NOT dispose the underlying stream |

### Reading Methods

#### ReadLine() -- One Line at a Time

The most common way to read a text file. Returns `null` at end-of-stream.

```csharp
using var reader = new StreamReader("log.txt");
string? line;

while ((line = reader.ReadLine()) != null)
{
    Console.WriteLine(line);
}
```

```ad-note
title: ReadLine() Strips the Newline
`ReadLine()` reads characters up to and including the newline (`\n` or `\r\n`), but the returned string does **not** include the newline characters. If you need to preserve line endings, use `Read()` or `ReadBlock()` instead.
```

#### ReadToEnd() -- Entire File as One String

Reads from the **current position** to the end of the stream, returning everything as a single string.

```csharp
using var reader = new StreamReader("config.json");
string entireFile = reader.ReadToEnd();
```

```ad-warning
title: ReadToEnd() Loads Everything into Memory
For large files, `ReadToEnd()` allocates a single string containing the entire file contents. This can cause `OutOfMemoryException` for very large files. Use `ReadLine()` in a loop for large files.
```

#### Read() -- Single Character

Returns the next character as an `int` (cast to `char`), or `-1` at end-of-stream.

```csharp
int ch;
while ((ch = reader.Read()) != -1)
{
    char c = (char)ch;
    // process character
}
```

#### Read(char[], int, int) / ReadBlock(char[], int, int)

Read multiple characters into a buffer.

- `Read()` may return fewer characters than requested (like `Stream.Read()`)
- `ReadBlock()` **blocks** until it reads exactly the requested count, or reaches end-of-stream

```csharp
char[] buffer = new char[1024];

// May return fewer than 1024
int charsRead = reader.Read(buffer, 0, buffer.Length);

// Guaranteed to read 1024 (or all remaining if fewer)
int charsBlock = reader.ReadBlock(buffer, 0, buffer.Length);
```

#### Peek() -- Look Ahead Without Consuming

Returns the next character without advancing the position. Returns `-1` at end-of-stream.

```csharp
if (reader.Peek() == '{')
{
    // next character is '{' -- probably JSON
    string json = reader.ReadToEnd();
}
```

#### EndOfStream Property

`true` when the reader has reached the end of the underlying stream.

```csharp
while (!reader.EndOfStream)
{
    string? line = reader.ReadLine();
    // process line
}
```

### Async Reading Methods

| Synchronous | Asynchronous | Returns |
|---|---|---|
| `ReadLine()` | `ReadLineAsync()` | `Task<string?>` |
| `ReadToEnd()` | `ReadToEndAsync()` | `Task<string>` |
| `Read(char[], int, int)` | `ReadAsync(char[], int, int)` | `Task<int>` |
| `ReadBlock(char[], int, int)` | `ReadBlockAsync(char[], int, int)` | `Task<int>` |

```csharp
using var reader = new StreamReader("log.txt");
string? line;

while ((line = await reader.ReadLineAsync()) != null)
{
    await ProcessLineAsync(line);
}
```

---

## StreamWriter -- Writing Text

### Creating a StreamWriter

```csharp
// To a file path (creates or overwrites)
using var writer1 = new StreamWriter("output.txt");

// To a file path, appending
using var writer2 = new StreamWriter("log.txt", append: true);

// To a file path with encoding
using var writer3 = new StreamWriter("output.txt", append: false, Encoding.UTF8);

// Wrapping an existing stream
using var fs = new FileStream("output.txt", FileMode.Create);
using var writer4 = new StreamWriter(fs);

// Wrapping with leaveOpen
using var writer5 = new StreamWriter(fs, Encoding.UTF8, bufferSize: 1024, leaveOpen: true);

// Via File convenience methods
using StreamWriter writer6 = File.CreateText("output.txt");     // create or overwrite
using StreamWriter writer7 = File.AppendText("log.txt");        // append
```

### Key Constructor Parameters

| Parameter | Default | Description |
|---|---|---|
| `encoding` | UTF-8 (no BOM in .NET Core; with BOM in .NET Framework) | Encoding for converting chars to bytes |
| `bufferSize` | 1024 | Internal character buffer size |
| `leaveOpen` | `false` | If `true`, disposing the writer does NOT dispose the underlying stream |
| `append` | `false` | (path constructor only) `true` to append, `false` to overwrite |

### Writing Methods

#### Write() and WriteLine()

```csharp
using var writer = new StreamWriter("output.txt");

// Write without a newline
writer.Write("Hello");
writer.Write(" ");
writer.Write("World");
// File now contains: Hello World (no newline)

// Write with a trailing newline
writer.WriteLine("First line");
writer.WriteLine("Second line");
// File now contains:
// First line
// Second line
```

#### Write Overloads for Various Types

`Write()` and `WriteLine()` accept many types -- they call `.ToString()` internally:

```csharp
writer.Write(42);           // writes "42"
writer.Write(3.14);         // writes "3.14"
writer.Write(true);         // writes "True"
writer.Write('A');          // writes "A"
writer.Write(DateTime.Now); // writes the string representation
```

#### String Interpolation and Formatting

```csharp
string name = "Long";
int age = 28;

writer.WriteLine($"Name: {name}, Age: {age}");
writer.WriteLine("Pi = {0:F4}", Math.PI); // formatted: "Pi = 3.1416"
```

### Flush() and AutoFlush

`StreamWriter` buffers data internally for performance. Data may not be written to the underlying stream until:

1. The internal buffer is full
2. You call `Flush()` manually
3. The `StreamWriter` is disposed

```csharp
writer.WriteLine("Important log entry");
writer.Flush(); // force the data to the underlying stream immediately
```

#### AutoFlush Property

When `true`, `StreamWriter` flushes after **every write operation**. This is slower but ensures no data is lost if the application crashes.

```csharp
using var writer = new StreamWriter("realtime.log");
writer.AutoFlush = true; // every Write/WriteLine immediately flushes

writer.WriteLine("Event 1"); // immediately written to disk
writer.WriteLine("Event 2"); // immediately written to disk
```

```ad-note
title: When to Use AutoFlush
Set `AutoFlush = true` for **logging** or **real-time monitoring** scenarios where you need every line written immediately. For **bulk data writing** (generating reports, exporting data), leave it `false` for significantly better performance -- the default buffering reduces the number of I/O operations.
```

### Async Writing Methods

| Synchronous | Asynchronous |
|---|---|
| `Write(string)` | `WriteAsync(string)` |
| `WriteLine(string)` | `WriteLineAsync(string)` |
| `Flush()` | `FlushAsync()` |

```csharp
using var writer = new StreamWriter("output.txt");

await writer.WriteLineAsync("Async line 1");
await writer.WriteLineAsync("Async line 2");
await writer.FlushAsync();
```

---

## Appending to Files

A common task is adding content to the end of an existing file without overwriting it.

```csharp
// Option 1: StreamWriter with append: true
using var writer = new StreamWriter("log.txt", append: true);
writer.WriteLine($"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] Application started");

// Option 2: File.AppendText
using var writer2 = File.AppendText("log.txt");
writer2.WriteLine($"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] Event occurred");

// Option 3: One-liner for simple appends
File.AppendAllText("log.txt", $"[{DateTime.Now:yyyy-MM-dd HH:mm:ss}] Quick note\n");
```

---

## Encoding -- Converting Between Bytes and Characters

Encoding is the mapping between bytes (what the stream holds) and characters (what your code works with). Getting the encoding wrong produces garbled text, not exceptions.

### Common Encodings

| Encoding | Class | Bytes per Char | Description |
|---|---|---|---|
| UTF-8 | `Encoding.UTF8` | 1-4 | Default in .NET Core/5+. Variable-width. ASCII-compatible for the first 128 chars. Most common encoding on the web and in modern files. |
| UTF-16 LE | `Encoding.Unicode` | 2 (or 4 for surrogates) | .NET's internal string encoding. Used by Windows APIs. Two bytes per character minimum. |
| UTF-16 BE | `Encoding.BigEndianUnicode` | 2 (or 4) | Same as UTF-16 but big-endian byte order. |
| ASCII | `Encoding.ASCII` | 1 | 7-bit, 0-127 only. Characters above 127 become `?`. |
| Latin-1 (ISO-8859-1) | `Encoding.Latin1` | 1 | 8-bit, 0-255. Covers Western European characters. |
| UTF-32 | `Encoding.UTF32` | 4 | Fixed 4 bytes per character. Rarely used for files. |

```csharp
// Specify encoding explicitly
using var reader = new StreamReader("data.txt", Encoding.ASCII);
using var writer = new StreamWriter("output.txt", false, Encoding.Unicode);
```

### Byte Order Mark (BOM)

A BOM is a special byte sequence at the start of a file that identifies its encoding:

| Encoding | BOM Bytes |
|---|---|
| UTF-8 | `EF BB BF` |
| UTF-16 LE | `FF FE` |
| UTF-16 BE | `FE FF` |
| UTF-32 LE | `FF FE 00 00` |

```ad-info
title: BOM Detection
By default, `StreamReader` has `detectEncodingFromByteOrderMarks: true`, which means it reads the first few bytes to check for a BOM and automatically selects the correct encoding. If no BOM is found, it falls back to the encoding you specified (or UTF-8 by default).
```

### The UTF-8 BOM Difference Between .NET Framework and .NET Core

```ad-warning
title: .NET Framework vs .NET Core/5+ BOM Behavior
- **.NET Framework**: `new StreamWriter(path)` writes a UTF-8 BOM (`EF BB BF`) at the start of the file.
- **.NET Core/5+**: `new StreamWriter(path)` writes UTF-8 **without** a BOM.

If you need consistent behavior, specify the encoding explicitly:
- No BOM: `new StreamWriter(path, false, new UTF8Encoding(encoderShouldEmitUTF8Identifier: false))`
- With BOM: `new StreamWriter(path, false, new UTF8Encoding(encoderShouldEmitUTF8Identifier: true))`
```

### Mojibake -- What Happens with the Wrong Encoding

```ad-warning
title: Wrong Encoding = Garbled Text, Not Errors
If you read a UTF-8 file with `Encoding.ASCII`, characters above 127 silently become `?`. If you read a Latin-1 file as UTF-8, multi-byte sequences get misinterpreted. You get **garbled text** (mojibake), not an exception. Always know the encoding of your files.
```

```csharp
// Write UTF-8 with non-ASCII characters
File.WriteAllText("test.txt", "café naif̈ve", Encoding.UTF8);

// Read with wrong encoding -- garbled output
using var wrongReader = new StreamReader("test.txt", Encoding.ASCII);
string garbled = wrongReader.ReadToEnd(); // "caf?? nai??ve" or similar garbage
```

---

## The leaveOpen Parameter

When a `StreamReader` or `StreamWriter` is disposed, it **also disposes the underlying stream** by default. The `leaveOpen` parameter overrides this behavior.

```csharp
// Default behavior: disposing writer also disposes the stream
using var ms = new MemoryStream();
using (var writer = new StreamWriter(ms)) // leaveOpen: false by default
{
    writer.WriteLine("Hello");
} // writer disposed → ms is ALSO disposed here

// ms is now disposed -- this throws ObjectDisposedException:
// ms.Position = 0; // ERROR!
```

```csharp
// With leaveOpen: true -- stream survives writer disposal
using var ms = new MemoryStream();

using (var writer = new StreamWriter(ms, Encoding.UTF8, bufferSize: 1024, leaveOpen: true))
{
    writer.WriteLine("Hello");
} // writer disposed, but ms stays open

ms.Position = 0; // OK -- stream is still alive
using var reader = new StreamReader(ms);
string content = reader.ReadToEnd(); // "Hello\r\n"
```

```ad-important
title: When to Use leaveOpen
Use `leaveOpen: true` when you need to continue using the underlying stream after the reader/writer is done -- commonly with [[MemoryStream]] where you write with a `StreamWriter`, then rewind and read with a `StreamReader`.
```

---

## StreamReader/StreamWriter vs File Convenience Methods

| Scenario | Recommended Approach | Why |
|---|---|---|
| Read entire small file as string | `File.ReadAllText("file.txt")` | One-liner, handles open/read/close |
| Read all lines into string[] | `File.ReadAllLines("file.txt")` | Convenient for small files |
| Read line by line (large file) | `StreamReader` with `ReadLine()` loop, or `File.ReadLines()` | Does not load entire file into memory |
| Write entire string to file | `File.WriteAllText("file.txt", text)` | One-liner, handles create/write/close |
| Write string[] as lines | `File.WriteAllLines("file.txt", lines)` | Convenient for small data |
| Append text | `File.AppendAllText()` or `StreamWriter(path, append: true)` | One-liner for simple appends; `StreamWriter` for repeated appends |
| Custom encoding | `StreamReader(path, encoding)` / `StreamWriter(path, false, encoding)` | `File` methods also accept encoding, but `StreamReader`/`StreamWriter` give more control |
| Write many lines in a loop | `StreamWriter` | Open once, write many times -- much more efficient than repeated `File.AppendAllText` calls |
| Wrap a non-file stream | `StreamReader(stream)` / `StreamWriter(stream)` | `File` methods only work with file paths |

```ad-note
title: File.ReadLines() vs File.ReadAllLines()
`File.ReadAllLines()` reads the **entire file into a `string[]`** at once -- the whole file must fit in memory.

`File.ReadLines()` returns an `IEnumerable<string>` that reads **lazily** -- lines are read one at a time as you enumerate. Use this for large files.
```

---

## Practical Example -- Log File Parser

```csharp
public static IEnumerable<LogEntry> ParseLogFile(string path)
{
    using var reader = new StreamReader(path, Encoding.UTF8);
    string? line;
    int lineNumber = 0;

    while ((line = reader.ReadLine()) != null)
    {
        lineNumber++;

        // Skip empty lines and comments
        if (string.IsNullOrWhiteSpace(line) || line.StartsWith('#'))
            continue;

        // Expected format: [2026-06-03 14:30:00] INFO Message text here
        if (TryParseLogLine(line, out var entry))
        {
            yield return entry;
        }
        else
        {
            Console.Error.WriteLine($"Warning: Could not parse line {lineNumber}: {line}");
        }
    }
}
```

### Practical Example -- CSV Writer

```csharp
public static void WriteCsv<T>(string path, IEnumerable<T> records, string[] headers)
{
    using var writer = new StreamWriter(path, append: false, Encoding.UTF8);

    // Write header row
    writer.WriteLine(string.Join(",", headers));

    // Write data rows
    foreach (T record in records)
    {
        // Use reflection or a mapping function to get field values
        string[] values = GetFieldValues(record, headers);

        // Escape values containing commas or quotes
        string[] escaped = values.Select(v =>
            v.Contains(',') || v.Contains('"')
                ? $"\"{v.Replace("\"", "\"\"")}\""
                : v
        ).ToArray();

        writer.WriteLine(string.Join(",", escaped));
    }
}
```

---

## StringReader and StringWriter -- In-Memory Text I/O

`StringReader` and `StringWriter` are the text equivalents of [[MemoryStream]] -- they work with in-memory strings instead of streams. They inherit from `TextReader`/`TextWriter` (the same base as `StreamReader`/`StreamWriter`).

```csharp
// StringReader -- read from a string as if it were a file
string data = "Line 1\nLine 2\nLine 3";
using var reader = new StringReader(data);
string? line;
while ((line = reader.ReadLine()) != null)
{
    Console.WriteLine(line);
}

// StringWriter -- build a string via writer methods
using var writer = new StringWriter();
writer.WriteLine("First line");
writer.WriteLine("Second line");
writer.Write("Partial ");
writer.Write("line");
string result = writer.ToString();
// result: "First line\r\nSecond line\r\nPartial line"
```

These are useful for unit testing code that accepts `TextReader`/`TextWriter` parameters, or for in-memory text processing.

---

## Summary

| Concept | Detail |
|---|---|
| What they are | Text adapters layered on top of a [[Stream]] -- handle encoding between bytes and chars |
| StreamReader methods | `ReadLine()`, `ReadToEnd()`, `Read()`, `ReadBlock()`, `Peek()`, `EndOfStream` |
| StreamWriter methods | `Write()`, `WriteLine()`, `Flush()`, `AutoFlush` property |
| Encoding | Default UTF-8. Always specify explicitly when interoperating with external systems. Wrong encoding = garbled text, not errors. |
| leaveOpen | Set `true` when you need the underlying stream to survive disposal of the reader/writer |
| Async | `ReadLineAsync()`, `ReadToEndAsync()`, `WriteAsync()`, `WriteLineAsync()`, `FlushAsync()` |
| AutoFlush | `true` for logging (immediate writes), `false` (default) for bulk writes (better performance) |
| Convenience alternatives | `File.ReadAllText()`, `File.ReadLines()`, `File.WriteAllText()`, `File.AppendAllText()` |
| In-memory variants | `StringReader` / `StringWriter` for working with strings instead of streams |
