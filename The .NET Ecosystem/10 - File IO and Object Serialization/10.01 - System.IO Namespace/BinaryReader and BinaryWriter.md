---
tags:
 - csharp
 - io
 - binary
---

## BinaryReader and BinaryWriter -- Primitive Type I/O in Binary Format

`BinaryReader` and `BinaryWriter` are adapters that sit on top of a [[Stream]] and read/write **.NET primitive types in their raw binary representation**. Unlike [[StreamReader and StreamWriter]] which convert everything to human-readable text, `BinaryWriter` writes the actual bytes that represent a value in memory.

```
┌──────────────────────────────────────────────────┐
│  Your Code                                       │
│  writer.Write(42)  /  reader.ReadInt32()         │
├──────────────────────────────────────────────────┤
│  BinaryWriter / BinaryReader                     │
│  (serializes/deserializes .NET primitives)       │
│  (int → 4 bytes, double → 8 bytes, etc.)        │
├──────────────────────────────────────────────────┤
│  Stream (FileStream, MemoryStream, etc.)         │
│  (raw byte[] I/O)                                │
└──────────────────────────────────────────────────┘
```

**Key differences from text I/O:**

| Aspect | BinaryReader/Writer | StreamReader/Writer |
|---|---|---|
| Format | Raw bytes -- not human-readable | Human-readable text |
| Size | Compact (an `int` is always 4 bytes) | Variable (the `int` 1000000 is 7 text characters) |
| Speed | Faster (no parsing/formatting overhead) | Slower (must parse text ↔ numbers) |
| Portability | .NET-specific binary format | Universal text format |
| Debugging | Cannot open in a text editor | Can open in any text editor |

---

## BinaryWriter -- Writing Primitive Types

### Creating a BinaryWriter

```csharp
// Wrap a FileStream
using var fs = new FileStream("data.bin", FileMode.Create, FileAccess.Write);
using var writer = new BinaryWriter(fs);

// Wrap a FileStream with explicit encoding (affects string writes only)
using var writer2 = new BinaryWriter(fs, Encoding.UTF8);

// Wrap with leaveOpen -- stream survives writer disposal
using var writer3 = new BinaryWriter(fs, Encoding.UTF8, leaveOpen: true);

// Wrap a MemoryStream for in-memory binary building
using var ms = new MemoryStream();
using var writer4 = new BinaryWriter(ms, Encoding.UTF8, leaveOpen: true);
```

### Write() Method Overloads

`BinaryWriter.Write()` has overloads for every primitive type. Each writes the value in its exact binary representation using **little-endian** byte order (least significant byte first).

```csharp
using var fs = new FileStream("data.bin", FileMode.Create);
using var writer = new BinaryWriter(fs, Encoding.UTF8);

writer.Write(42);               // int    → 4 bytes:  2A 00 00 00
writer.Write(3.14);             // double → 8 bytes:  1F 85 EB 51 B8 1E 09 40
writer.Write(3.14f);            // float  → 4 bytes:  C3 F5 48 40
writer.Write((short)1000);      // short  → 2 bytes:  E8 03
writer.Write((long)123456789L); // long   → 8 bytes:  15 CD 5B 07 00 00 00 00
writer.Write(true);             // bool   → 1 byte:   01
writer.Write(false);            // bool   → 1 byte:   00
writer.Write((byte)0xFF);       // byte   → 1 byte:   FF
writer.Write('A');              // char   → 1+ bytes: 41 (UTF-8 encoded)
writer.Write(99.99m);           // decimal→ 16 bytes
writer.Write("Hello");          // string → length-prefixed UTF-8 (see below)
```

### How Strings Are Written

Strings are written as a **length-prefixed** sequence of encoded bytes:

1. First, the byte length of the encoded string is written as a **7-bit encoded integer** (variable length: 1 byte for lengths 0-127, 2 bytes for 128-16383, etc.)
2. Then, the actual string bytes are written using the specified encoding (default UTF-8)

```csharp
writer.Write("Hello");
// Writes: 05 48 65 6C 6C 6F
//         ^^ ^^^^^^^^^^^^^^^^
//         |  "Hello" in UTF-8 (5 bytes)
//         Length prefix: 5

writer.Write(""); // Writes: 00 (just the length prefix: 0)
```

```ad-note
title: The Length Prefix Is NOT a Fixed Size
The length prefix uses a compact variable-length encoding (LEB128 / 7-bit encoded int). Strings up to 127 bytes use a 1-byte prefix, strings up to 16383 bytes use 2 bytes, and so on. This means you cannot calculate file offsets by simple arithmetic unless you know the encoded string lengths in advance.
```

### Writing Raw Byte Arrays

```csharp
byte[] rawData = { 0x00, 0x01, 0x02, 0x03 };
writer.Write(rawData);                  // writes all 4 bytes
writer.Write(rawData, 1, 2);            // writes bytes at index 1 and 2: 01 02
```

```ad-warning
title: Write(byte[]) Has No Length Prefix
When you write a `byte[]` with `BinaryWriter.Write(byte[])`, it writes the raw bytes **without a length prefix**. When reading back, you must already know how many bytes to read. This is different from `Write(string)`, which includes the length.
```

### Accessing the Underlying Stream

```csharp
// The BaseStream property gives direct access to the underlying stream
writer.BaseStream.Position; // current write position
writer.BaseStream.Length;   // total bytes written so far (for MemoryStream)
```

---

## BinaryReader -- Reading Primitive Types

### Creating a BinaryReader

```csharp
// From a FileStream
using var fs = new FileStream("data.bin", FileMode.Open, FileAccess.Read);
using var reader = new BinaryReader(fs);

// With explicit encoding (must match what was used to write)
using var reader2 = new BinaryReader(fs, Encoding.UTF8);

// With leaveOpen
using var reader3 = new BinaryReader(fs, Encoding.UTF8, leaveOpen: true);
```

### Read Methods -- Complete Reference

Each `Read*` method reads the exact number of bytes for that type and advances the stream position accordingly.

| Method | .NET Type | Bytes Read | Value Range |
|---|---|---|---|
| `ReadBoolean()` | `bool` | 1 | `true` (non-zero) / `false` (0) |
| `ReadByte()` | `byte` | 1 | 0 to 255 |
| `ReadSByte()` | `sbyte` | 1 | -128 to 127 |
| `ReadChar()` | `char` | 1-4 (encoding) | Single Unicode character |
| `ReadInt16()` | `short` | 2 | -32,768 to 32,767 |
| `ReadUInt16()` | `ushort` | 2 | 0 to 65,535 |
| `ReadInt32()` | `int` | 4 | -2,147,483,648 to 2,147,483,647 |
| `ReadUInt32()` | `uint` | 4 | 0 to 4,294,967,295 |
| `ReadInt64()` | `long` | 8 | -9.2 x 10^18 to 9.2 x 10^18 |
| `ReadUInt64()` | `ulong` | 8 | 0 to 1.8 x 10^19 |
| `ReadHalf()` | `Half` | 2 | (.NET 5+) 16-bit floating point |
| `ReadSingle()` | `float` | 4 | ~7 digits precision |
| `ReadDouble()` | `double` | 8 | ~15-16 digits precision |
| `ReadDecimal()` | `decimal` | 16 | 28-29 significant digits |
| `ReadString()` | `string` | variable | Length-prefixed encoded string |
| `ReadBytes(int count)` | `byte[]` | `count` | Raw byte array |

```csharp
using var fs = File.OpenRead("data.bin");
using var reader = new BinaryReader(fs, Encoding.UTF8);

int number = reader.ReadInt32();       // reads 4 bytes → int
double pi = reader.ReadDouble();       // reads 8 bytes → double
float ratio = reader.ReadSingle();     // reads 4 bytes → float
short small = reader.ReadInt16();      // reads 2 bytes → short
long big = reader.ReadInt64();         // reads 8 bytes → long
bool flag = reader.ReadBoolean();      // reads 1 byte  → bool
byte b = reader.ReadByte();            // reads 1 byte  → byte
char c = reader.ReadChar();            // reads 1+ bytes → char
decimal money = reader.ReadDecimal();  // reads 16 bytes → decimal
string text = reader.ReadString();     // reads length prefix + encoded bytes → string
byte[] raw = reader.ReadBytes(100);    // reads exactly 100 bytes
```

---

## The Cardinal Rule: Read in the Same Order You Wrote

```ad-warning
title: Order and Types Must Match Exactly
Binary files have **no metadata, no headers, no type tags**. The file is just a flat sequence of bytes. `BinaryReader` has no way to know what type of data is at the current position -- it just reads the number of bytes you tell it to and reinterprets them.

==If you read in a different order or with a different type than you wrote, you get meaningless garbage, not an error.==
```

```csharp
// WRITE: int, double, string
using (var writer = new BinaryWriter(File.Create("data.bin")))
{
    writer.Write(42);       // 4 bytes
    writer.Write(3.14);     // 8 bytes
    writer.Write("Hello");  // 6 bytes (1 length + 5 chars)
}

// CORRECT READ: same order, same types
using (var reader = new BinaryReader(File.OpenRead("data.bin")))
{
    int n = reader.ReadInt32();      // reads bytes 0-3 as int → 42
    double d = reader.ReadDouble();  // reads bytes 4-11 as double → 3.14
    string s = reader.ReadString();  // reads bytes 12-17 as string → "Hello"
}

// WRONG READ: different order -- data is scrambled
using (var reader = new BinaryReader(File.OpenRead("data.bin")))
{
    double d = reader.ReadDouble();  // reads bytes 0-7 as double → garbage!
    int n = reader.ReadInt32();      // reads bytes 8-11 as int → garbage!
    string s = reader.ReadString();  // tries to read length prefix → unpredictable
}
```

### EndOfStreamException

If you try to read past the end of the stream, `BinaryReader` throws `EndOfStreamException`:

```csharp
using var reader = new BinaryReader(File.OpenRead("small.bin")); // file has 4 bytes
int a = reader.ReadInt32();  // OK -- reads 4 bytes
int b = reader.ReadInt32();  // throws EndOfStreamException -- no more bytes
```

You can check the stream's position and length to avoid this:

```csharp
while (reader.BaseStream.Position < reader.BaseStream.Length)
{
    // safe to read more data
}
```

---

## Practical Example -- Saving and Loading a Record

A common pattern is writing a structured record to a binary file and reading it back.

```csharp
public record Person(string Name, int Age, double Height, bool IsActive);

public static class PersonStorage
{
    public static void Save(string path, Person person)
    {
        using var fs = new FileStream(path, FileMode.Create, FileAccess.Write);
        using var writer = new BinaryWriter(fs, Encoding.UTF8);

        writer.Write(person.Name);      // length-prefixed string
        writer.Write(person.Age);       // 4 bytes
        writer.Write(person.Height);    // 8 bytes
        writer.Write(person.IsActive);  // 1 byte
    }

    public static Person Load(string path)
    {
        using var fs = new FileStream(path, FileMode.Open, FileAccess.Read);
        using var reader = new BinaryReader(fs, Encoding.UTF8);

        string name = reader.ReadString();
        int age = reader.ReadInt32();
        double height = reader.ReadDouble();
        bool isActive = reader.ReadBoolean();

        return new Person(name, age, height, isActive);
    }
}

// Usage
PersonStorage.Save("person.dat", new Person("Long", 28, 1.75, true));
Person loaded = PersonStorage.Load("person.dat");
Console.WriteLine(loaded); // Person { Name = Long, Age = 28, Height = 1.75, IsActive = True }
```

---

## Practical Example -- Saving and Loading a Collection

When writing a collection, you must write the **count** first so the reader knows how many items to expect.

```csharp
public static void SaveScores(string path, List<(string Name, int Score)> scores)
{
    using var writer = new BinaryWriter(File.Create(path), Encoding.UTF8);

    writer.Write(scores.Count); // write count first

    foreach (var (name, score) in scores)
    {
        writer.Write(name);
        writer.Write(score);
    }
}

public static List<(string Name, int Score)> LoadScores(string path)
{
    using var reader = new BinaryReader(File.OpenRead(path), Encoding.UTF8);

    int count = reader.ReadInt32(); // read count first
    var scores = new List<(string, int)>(count);

    for (int i = 0; i < count; i++)
    {
        string name = reader.ReadString();
        int score = reader.ReadInt32();
        scores.Add((name, score));
    }

    return scores;
}

// Usage
var data = new List<(string, int)>
{
    ("Alice", 95),
    ("Bob", 87),
    ("Charlie", 92)
};

SaveScores("scores.dat", data);
var loaded = LoadScores("scores.dat");
// loaded contains the same 3 entries
```

---

## Practical Example -- File Format with a Magic Number Header

Real binary file formats typically start with a **magic number** (a fixed byte sequence) and a **version number** for forward compatibility.

```csharp
public static class CustomFormat
{
    private static readonly byte[] MagicNumber = { 0x4D, 0x59, 0x46, 0x4D }; // "MYFM"
    private const int FormatVersion = 1;

    public static void WriteFile(string path, string title, byte[] payload)
    {
        using var writer = new BinaryWriter(File.Create(path), Encoding.UTF8);

        // Header
        writer.Write(MagicNumber);           // 4 bytes -- magic number
        writer.Write(FormatVersion);          // 4 bytes -- version
        writer.Write(title);                  // variable -- length-prefixed string
        writer.Write(payload.Length);          // 4 bytes -- payload size
        writer.Write(payload);                // N bytes -- raw payload
    }

    public static (string Title, byte[] Payload) ReadFile(string path)
    {
        using var reader = new BinaryReader(File.OpenRead(path), Encoding.UTF8);

        // Validate magic number
        byte[] magic = reader.ReadBytes(4);
        if (!magic.SequenceEqual(MagicNumber))
            throw new InvalidDataException("Not a valid MYFM file");

        // Check version
        int version = reader.ReadInt32();
        if (version > FormatVersion)
            throw new InvalidDataException($"Unsupported format version: {version}");

        string title = reader.ReadString();
        int payloadLength = reader.ReadInt32();
        byte[] payload = reader.ReadBytes(payloadLength);

        return (title, payload);
    }
}
```

---

## Binary vs Text -- When to Choose Which

| Criteria | Binary (BinaryReader/Writer) | Text (StreamReader/Writer) |
|---|---|---|
| **File size** | Compact -- `int` 42 = 4 bytes always | Larger -- `int` 42 = 2 characters + encoding |
| **Speed** | Faster -- no parsing/formatting | Slower -- must convert between text and types |
| **Human readable** | No -- looks like gibberish in a text editor | Yes -- can inspect and edit in any editor |
| **Debugging** | Hard -- need a hex editor | Easy -- open in Notepad |
| **Interoperability** | .NET-specific format (endianness, string encoding) | Universal -- any language can read text/CSV/JSON |
| **Precision** | Exact -- `double` preserves all 64 bits | May lose precision in text round-trip (`3.14159265358979` vs `3.14159265358979323846`) |
| **Schema evolution** | Fragile -- adding a field breaks old readers | More flexible with named formats (JSON, XML) |
| **Best for** | Internal storage, performance-critical data, games, custom file formats | Config files, logs, data exchange, APIs, anything humans need to read |

```ad-info
title: When to Use Neither
For complex object graphs, nested structures, or cross-platform data exchange, consider structured serialization formats like JSON (`System.Text.Json`), XML, Protocol Buffers, or MessagePack. `BinaryReader`/`BinaryWriter` are best for simple flat records or custom binary protocols.
```

---

## Byte Order (Endianness)

`BinaryWriter` always writes in **little-endian** format (least significant byte first), which is the native byte order on x86/x64 and ARM processors running Windows and Linux.

```csharp
writer.Write((int)256);
// Writes: 00 01 00 00 (little-endian)
// NOT:    00 00 01 00 (big-endian)
```

```ad-warning
title: Endianness Matters for Cross-Platform Binary Files
If you're reading binary data from a system that uses **big-endian** byte order (some network protocols, Java's `DataOutputStream`, some hardware), you need to reverse the bytes manually or use `BinaryPrimitives` (in `System.Buffers.Binary`):

```csharp
using System.Buffers.Binary;

byte[] bigEndianBytes = { 0x00, 0x00, 0x01, 0x00 }; // 256 in big-endian
int value = BinaryPrimitives.ReadInt32BigEndian(bigEndianBytes); // 256
```
```

---

## Summary

| Concept | Detail |
|---|---|
| What they are | Adapters over a [[Stream]] that read/write .NET primitives in raw binary format |
| BinaryWriter.Write() | Overloads for `int`, `double`, `float`, `long`, `short`, `bool`, `byte`, `char`, `decimal`, `string`, `byte[]` |
| BinaryReader.Read*() | `ReadInt32()`, `ReadDouble()`, `ReadSingle()`, `ReadString()`, `ReadBoolean()`, `ReadBytes(n)`, etc. |
| Strings | Written as length-prefixed UTF-8 (or specified encoding). `ReadString()` / `Write(string)` |
| Cardinal rule | ==Read in the exact same order and types as you wrote== -- there is no metadata |
| Byte order | Always **little-endian**. Use `BinaryPrimitives` for big-endian interop. |
| Collections | Write the count first, then each element. Read count, loop that many times. |
| When to use | Internal storage, performance-critical data, custom file formats, game save files |
| When NOT to use | Human-readable data, cross-platform exchange, complex nested objects -- use JSON/XML instead |
