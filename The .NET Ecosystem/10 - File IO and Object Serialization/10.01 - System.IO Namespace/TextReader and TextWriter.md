---
tags:
 - csharp
 - io
 - text-io
---

## TextReader and TextWriter -- Abstract Base Classes for All Text I/O

`TextReader` and `TextWriter` are the **abstract base classes** for all character-based I/O in .NET. They define a common API for reading and writing text, regardless of whether the backing store is a file on disk, an in-memory string, or something else entirely.

The key insight is **polymorphism**: any code that accepts a `TextReader` parameter works with files, strings, or any future implementation -- without knowing or caring which one it is.

---

## Inheritance Hierarchy

```
TextReader (abstract)
├── StreamReader  -- reads text from a byte Stream (file, network, memory, etc.)
└── StringReader  -- reads text from an in-memory string

TextWriter (abstract)
├── StreamWriter  -- writes text to a byte Stream
└── StringWriter  -- writes text to an in-memory StringBuilder
```

`TextReader`/`TextWriter` are in the `System.IO` namespace. They do **not** inherit from [[Stream]] -- they are a completely separate abstraction. `Stream` is for raw bytes; `TextReader`/`TextWriter` are for characters and strings.

---

## Why They Exist -- Programming Against Abstractions

The primary reason these base classes exist is to enable **polymorphic text processing**. You write methods that accept `TextReader` or `TextWriter`, and the caller decides where the text comes from or goes to.

```csharp
// This method works with ANY TextReader -- files, strings, network streams...
int CountLines(TextReader reader)
{
    int count = 0;
    while (reader.ReadLine() != null)
        count++;
    return count;
}

// Works with a file:
using var fileReader = new StreamReader("data.txt");
int fileLines = CountLines(fileReader);

// Works with a string (great for unit testing):
using var stringReader = new StringReader("line 1\nline 2\nline 3");
int stringLines = CountLines(stringReader);  // returns 3

// Works with a network stream:
using var netReader = new StreamReader(networkStream);
int netLines = CountLines(netReader);
```

This is especially valuable for **testability**. Instead of writing tests that need real files on disk, you pass a `StringReader` with test data.

```ad-note
title: Design Pattern
This is the **Dependency Inversion Principle** in action -- a concrete example of "program against abstractions, not implementations." You may also see it described as the Strategy or Template Method pattern depending on context.
```

---

## TextReader -- Key Methods

| Method | Return Type | Description |
|---|---|---|
| `Read()` | `int` | Reads the next character. Returns the `char` as an `int`, or `-1` at end-of-stream. |
| `Read(char[], int, int)` | `int` | Reads up to `count` characters into a buffer. May return fewer than requested. |
| `ReadBlock(char[], int, int)` | `int` | Like `Read()` but blocks until exactly `count` characters are read, or end-of-stream. |
| `ReadLine()` | `string?` | Reads one line of text. Returns `null` at end-of-stream. The returned string does **not** include the newline. |
| `ReadToEnd()` | `string` | Reads all remaining text from the current position to the end. |
| `Peek()` | `int` | Returns the next character without consuming it. Returns `-1` at end-of-stream. |

### Async Counterparts

| Synchronous | Asynchronous | Returns |
|---|---|---|
| `Read(char[], int, int)` | `ReadAsync(char[], int, int)` | `Task<int>` |
| `ReadBlock(char[], int, int)` | `ReadBlockAsync(char[], int, int)` | `Task<int>` |
| `ReadLine()` | `ReadLineAsync()` | `Task<string?>` |
| `ReadToEnd()` | `ReadToEndAsync()` | `Task<string>` |

```csharp
async Task ProcessTextAsync(TextReader reader)
{
    string? line;
    while ((line = await reader.ReadLineAsync()) != null)
    {
        await HandleLineAsync(line);
    }
}
```

---

## TextWriter -- Key Methods

| Method | Description |
|---|---|
| `Write(string)` | Writes a string (no newline appended) |
| `Write(char)` | Writes a single character |
| `Write(char[])` | Writes a character array |
| `Write(int)` / `Write(double)` / etc. | Writes the `.ToString()` of the value |
| `WriteLine(string)` | Writes a string followed by the `NewLine` sequence |
| `WriteLine()` | Writes just the `NewLine` sequence |
| `Flush()` | Flushes any buffered data to the underlying store |

### Async Counterparts

| Synchronous | Asynchronous |
|---|---|
| `Write(string)` | `WriteAsync(string)` |
| `Write(char)` | `WriteAsync(char)` |
| `WriteLine(string)` | `WriteLineAsync(string)` |
| `Flush()` | `FlushAsync()` |

### The NewLine Property

`TextWriter` has a `NewLine` property (default `"\r\n"` on Windows, `"\n"` on Linux/macOS) that controls what `WriteLine()` appends. You can change it:

```csharp
using var writer = new StreamWriter("unix_file.txt");
writer.NewLine = "\n";  // force Unix line endings regardless of OS
writer.WriteLine("Line 1");  // writes "Line 1\n", not "Line 1\r\n"
```

---

## TextReader.Null and TextWriter.Null -- The /dev/null Equivalents

`TextReader.Null` and `TextWriter.Null` are singleton instances that discard all data. They are the text equivalents of `Stream.Null`.

```csharp
// TextReader.Null -- always at end-of-stream
TextReader nullReader = TextReader.Null;
string? line = nullReader.ReadLine();  // always returns null
int ch = nullReader.Read();            // always returns -1

// TextWriter.Null -- silently discards everything
TextWriter nullWriter = TextWriter.Null;
nullWriter.WriteLine("This goes nowhere");  // silently discarded
nullWriter.Flush();                          // no-op
```

Use cases for null readers/writers:
- **Default parameter values** -- when a method requires a `TextWriter` for logging but you want to suppress output
- **Testing** -- verify a method runs without producing output
- **Conditional logging** -- swap between a real writer and `Null` based on a verbosity flag

```csharp
void Process(TextReader input, TextWriter log)
{
    log.WriteLine("Starting processing...");  // logged or discarded depending on caller
    string? line;
    while ((line = input.ReadLine()) != null)
    {
        // process...
        log.WriteLine($"Processed: {line}");
    }
}

// With logging:
Process(new StreamReader("data.txt"), new StreamWriter("process.log"));

// Without logging:
Process(new StreamReader("data.txt"), TextWriter.Null);
```

---

## Console.In and Console.Out

`Console.In` is a `TextReader` and `Console.Out` is a `TextWriter`. This means any method that accepts `TextReader`/`TextWriter` can read from or write to the console:

```csharp
// Console.In is a TextReader
int lineCount = CountLines(Console.In);  // counts lines from stdin

// Console.Out is a TextWriter
WriteReport(Console.Out);  // writes directly to the console
```

This is particularly useful for building command-line tools that support piping:

```csharp
// Read from stdin or a file
TextReader input = args.Length > 0
    ? new StreamReader(args[0])
    : Console.In;

using (input)
{
    string? line;
    while ((line = input.ReadLine()) != null)
        Console.WriteLine(line.ToUpper());
}
```

---

## Concrete Implementations at a Glance

### StreamReader / StreamWriter

The most commonly used implementations. They wrap a byte [[Stream]] and handle **encoding** (converting between bytes and characters). See [[StreamReader and StreamWriter]] for full details.

```csharp
// StreamReader: TextReader that reads from a Stream
using TextReader reader = new StreamReader("data.txt", Encoding.UTF8);

// StreamWriter: TextWriter that writes to a Stream
using TextWriter writer = new StreamWriter("output.txt", append: false, Encoding.UTF8);
```

### StringReader / StringWriter

In-memory text I/O. `StringReader` reads from a string, `StringWriter` writes to an internal `StringBuilder`.

```csharp
// StringReader: TextReader that reads from a string
string data = "line 1\nline 2\nline 3";
using var reader = new StringReader(data);
Console.WriteLine(reader.ReadLine());  // "line 1"
Console.WriteLine(reader.ReadLine());  // "line 2"

// StringWriter: TextWriter that writes to a StringBuilder
using var writer = new StringWriter();
writer.WriteLine("Hello");
writer.Write("World");
string result = writer.ToString();  // "Hello\r\nWorld"

// StringWriter exposes its underlying StringBuilder
StringBuilder sb = writer.GetStringBuilder();
```

```ad-note
title: StringWriter Uses StringBuilder Internally
`StringWriter.ToString()` returns the accumulated text. You can also access the underlying `StringBuilder` via `GetStringBuilder()` for direct manipulation. `StringReader`, on the other hand, reads from an immutable `string` and provides no way to modify the source.
```

---

## Practical Example -- Testable Text Processing

This pattern demonstrates the real power of `TextReader`/`TextWriter`: writing processing logic that is easy to test without touching the file system.

```csharp
// Production code: process text from any source to any destination
public static void TransformCsv(TextReader input, TextWriter output)
{
    string? header = input.ReadLine();
    if (header == null) return;

    output.WriteLine(header.ToUpper());  // uppercase the header row

    string? line;
    while ((line = input.ReadLine()) != null)
    {
        string[] fields = line.Split(',');
        // Process fields...
        output.WriteLine(string.Join(",", fields));
    }
}

// In production -- reads from a file, writes to a file:
using var reader = new StreamReader("input.csv");
using var writer = new StreamWriter("output.csv");
TransformCsv(reader, writer);

// In a unit test -- no file system needed:
string testInput = "Name,Age\nAlice,30\nBob,25";
using var testReader = new StringReader(testInput);
using var testWriter = new StringWriter();

TransformCsv(testReader, testWriter);

string result = testWriter.ToString();
// result: "NAME,AGE\r\nAlice,30\r\nBob,25\r\n"
Assert.StartsWith("NAME,AGE", result);
```

---

## IDisposable -- Always Use `using`

Both `TextReader` and `TextWriter` implement `IDisposable`. For `StreamReader`/`StreamWriter`, disposal closes the underlying stream. For `StringReader`/`StringWriter`, disposal is lightweight (releases the internal reference) but you should still use `using` for consistency and to signal intent.

```ad-warning
title: Disposing a TextWriter Flushes Data
When you dispose a `StreamWriter`, it flushes buffered data to the underlying stream before closing. If you forget to dispose, ==buffered text may never be written==, causing silent data loss. Always use `using`.
```

---

## Summary

| Concept | Detail |
|---|---|
| What they are | Abstract base classes for all character-based I/O in .NET |
| TextReader subclasses | `StreamReader` (reads from a byte stream), `StringReader` (reads from a string) |
| TextWriter subclasses | `StreamWriter` (writes to a byte stream), `StringWriter` (writes to a StringBuilder) |
| Key TextReader methods | `Read()`, `ReadLine()`, `ReadToEnd()`, `ReadBlock()`, `Peek()`, plus async variants |
| Key TextWriter methods | `Write()`, `WriteLine()`, `Flush()`, `NewLine` property, plus async variants |
| Null singletons | `TextReader.Null` (always end-of-stream), `TextWriter.Null` (discards all output) |
| Console integration | `Console.In` is a `TextReader`, `Console.Out` is a `TextWriter` |
| Primary benefit | Polymorphism -- write once, read from files/strings/network/console with the same code |
| Testability | Accept `TextReader`/`TextWriter` in methods, then pass `StringReader`/`StringWriter` in tests |
