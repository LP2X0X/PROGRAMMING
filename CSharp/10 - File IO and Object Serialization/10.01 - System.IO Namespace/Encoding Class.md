---
tags:
 - csharp
 - io
 - encoding
---

## System.Text.Encoding -- Converting Between Characters and Bytes

**Encoding** is the mapping between human-readable characters (what your code works with as `char` and `string`) and raw byte sequences (what files, network sockets, and streams actually store). The `System.Text.Encoding` class is the abstract base that defines this mapping, and it provides static properties for every encoding you commonly need.

Understanding encoding is critical because ==every time a `StreamReader` or `StreamWriter` converts between text and bytes, an `Encoding` object does the actual work==. If you pick the wrong encoding, you get garbled text (mojibake), not an error.

---

## The Encoding Class -- Abstract Base with Static Accessors

`System.Text.Encoding` is an **abstract class** in the `System.Text` namespace. You rarely subclass it yourself -- instead, you access built-in encodings through its static properties.

```csharp
using System.Text;

Encoding utf8    = Encoding.UTF8;
Encoding unicode = Encoding.Unicode;
Encoding ascii   = Encoding.ASCII;
Encoding latin1  = Encoding.Latin1;
```

Each static property returns a **singleton instance** of the corresponding encoding. These are thread-safe and reusable.

---

## Common Encodings

| Encoding | Static Property | Bytes per Char | BOM | Use Case |
|---|---|---|---|---|
| UTF-8 | `Encoding.UTF8` | 1-4 | Optional (`EF BB BF`) | Default in .NET Core/5+, web, most modern files |
| UTF-16 LE | `Encoding.Unicode` | 2-4 | `FF FE` | Windows internal, .NET `string` representation |
| UTF-16 BE | `Encoding.BigEndianUnicode` | 2-4 | `FE FF` | Some network protocols |
| ASCII | `Encoding.ASCII` | 1 | None | Legacy, English-only (code points 0-127) |
| Latin-1 (ISO-8859-1) | `Encoding.Latin1` | 1 | None | Western European (code points 0-255) |
| UTF-32 LE | `Encoding.UTF32` | 4 | `FF FE 00 00` | Rare, fixed-width Unicode |

```ad-note
title: UTF-8 is Variable-Width
UTF-8 uses 1 byte for ASCII characters (0-127), 2 bytes for most European/Middle Eastern characters, 3 bytes for most CJK characters, and 4 bytes for emoji and rare symbols. This makes it compact for English text while still supporting the entire Unicode range.
```

```ad-note
title: .NET Strings Are Always UTF-16 Internally
Regardless of what encoding a file uses on disk, once text is loaded into a .NET `string`, it is stored as UTF-16 (`char` = 2-byte UTF-16 code unit). Encoding objects handle the conversion between the file's encoding and .NET's internal UTF-16 representation.
```

---

## Key Methods

### Encoding to Bytes

| Method | Description |
|---|---|
| `GetBytes(string)` | Converts the entire string to a `byte[]` |
| `GetBytes(char[], int, int)` | Converts a portion of a `char[]` to `byte[]` |
| `GetByteCount(string)` | Returns how many bytes the string would produce (without allocating) |
| `GetByteCount(char[], int, int)` | Returns how many bytes a char range would produce |

### Decoding to Characters/Strings

| Method | Description |
|---|---|
| `GetString(byte[])` | Converts a `byte[]` to a `string` |
| `GetString(byte[], int, int)` | Converts a portion of a `byte[]` to a `string` |
| `GetChars(byte[])` | Converts a `byte[]` to a `char[]` |
| `GetCharCount(byte[])` | Returns how many `char`s the bytes would produce |

### BOM

| Method | Description |
|---|---|
| `GetPreamble()` | Returns the BOM bytes for this encoding (empty array if none) |

---

## Code Examples

### Basic Encoding and Decoding

```csharp
using System.Text;

// UTF-8: ASCII characters use 1 byte each
byte[] utf8Bytes = Encoding.UTF8.GetBytes("Hello");       // [72, 101, 108, 108, 111]
string text      = Encoding.UTF8.GetString(utf8Bytes);     // "Hello"

// UTF-16 (Unicode): every character uses at least 2 bytes
byte[] unicodeBytes = Encoding.Unicode.GetBytes("Hello");  // [72, 0, 101, 0, 108, 0, 108, 0, 111, 0]
// Note the 0x00 bytes -- UTF-16 LE stores the low byte first

// ASCII: only handles 0-127
byte[] asciiBytes = Encoding.ASCII.GetBytes("cafe");       // [99, 97, 102, 101] -- 4 bytes
```

### Checking Byte Counts

```csharp
// UTF-8 byte count varies by character
Console.WriteLine(Encoding.UTF8.GetByteCount("Hello"));    // 5  (1 byte each)
Console.WriteLine(Encoding.UTF8.GetByteCount("cafe"));     // 5  (e with accent = 2 bytes)
Console.WriteLine(Encoding.UTF8.GetByteCount("Tokyo"));  // 15 (CJK = 3 bytes each)

// UTF-16 is always 2+ bytes per char
Console.WriteLine(Encoding.Unicode.GetByteCount("Hello")); // 10 (2 bytes each)
Console.WriteLine(Encoding.Unicode.GetByteCount("Tokyo")); // 10 (2 bytes each, within BMP)
```

### Encoding Non-ASCII Characters

```csharp
string text = "cafe naieve";

byte[] utf8  = Encoding.UTF8.GetBytes(text);    // e encoded as 2 bytes (C3 A9)
byte[] ascii = Encoding.ASCII.GetBytes(text);   // e becomes '?' (0x3F) -- data loss!
byte[] latin = Encoding.Latin1.GetBytes(text);  // e encoded as 1 byte (E9) -- works for Latin chars

Console.WriteLine(Encoding.UTF8.GetString(utf8));    // "cafe naieve" -- correct
Console.WriteLine(Encoding.ASCII.GetString(ascii));  // "caf? nai?ve" -- garbled
Console.WriteLine(Encoding.Latin1.GetString(latin)); // "cafe naieve" -- correct
```

```ad-warning
title: ASCII Silently Destroys Non-ASCII Characters
`Encoding.ASCII` replaces any character above code point 127 with `?` (0x3F). This is **silent data loss** -- no exception is thrown. Never use ASCII encoding unless you are 100% certain the data contains only 7-bit ASCII characters.
```

---

## BOM (Byte Order Mark)

A **BOM** is a special byte sequence placed at the very beginning of a file or byte stream that identifies the encoding. It solves the problem of "how do I know what encoding this file uses?"

| Encoding | BOM Bytes | Hex |
|---|---|---|
| UTF-8 | `EF BB BF` | 3 bytes |
| UTF-16 LE | `FF FE` | 2 bytes |
| UTF-16 BE | `FE FF` | 2 bytes |
| UTF-32 LE | `FF FE 00 00` | 4 bytes |

### Checking an Encoding's BOM

```csharp
byte[] bom = Encoding.UTF8.GetPreamble();
// bom = [0xEF, 0xBB, 0xBF]

byte[] unicodeBom = Encoding.Unicode.GetPreamble();
// unicodeBom = [0xFF, 0xFE]

byte[] asciiBom = Encoding.ASCII.GetPreamble();
// asciiBom = [] -- ASCII has no BOM
```

### BOM Behavior in .NET Framework vs .NET Core/5+

```ad-warning
title: Framework vs Core BOM Difference
- **.NET Framework**: `new StreamWriter(path)` writes a **UTF-8 BOM** (`EF BB BF`) at the start of the file by default.
- **.NET Core/5+**: `new StreamWriter(path)` writes UTF-8 **without** a BOM by default.

This can cause subtle bugs when migrating between frameworks, or when tools expect (or reject) a BOM.
```

To explicitly control BOM output:

```csharp
// Force BOM (both frameworks)
var withBom = new UTF8Encoding(encoderShouldEmitUTF8Identifier: true);
using var writer1 = new StreamWriter("with_bom.txt", false, withBom);

// Force no BOM (both frameworks)
var noBom = new UTF8Encoding(encoderShouldEmitUTF8Identifier: false);
using var writer2 = new StreamWriter("no_bom.txt", false, noBom);
```

```ad-note
title: BOM Detection by StreamReader
By default, `StreamReader` has `detectEncodingFromByteOrderMarks: true`. It reads the first few bytes to check for a BOM and auto-selects the encoding. If no BOM is found, it falls back to the encoding you specified (or UTF-8).
```

---

## Mojibake -- What Goes Wrong with the Wrong Encoding

**Mojibake** is the garbled text you get when bytes are decoded with the wrong encoding. It is one of the most common text-handling bugs, and it is insidious because ==reading a file with the wrong encoding produces garbage output, not an error==.

```csharp
// Write a file in UTF-8
File.WriteAllText("test.txt", "cafe naieve -- 5.00EUR", Encoding.UTF8);

// Read with the wrong encoding
string garbled = File.ReadAllText("test.txt", Encoding.ASCII);
// Result: "caf? nai?ve -- ?5.00" -- accented characters destroyed

// Read with the correct encoding
string correct = File.ReadAllText("test.txt", Encoding.UTF8);
// Result: "cafe naieve -- 5.00EUR"
```

Common mojibake scenarios:
- Reading a UTF-8 file as ASCII -- non-ASCII characters become `?`
- Reading a UTF-8 file as Latin-1 -- multi-byte sequences split into separate garbage characters
- Reading a Latin-1 file as UTF-8 -- bytes above 127 may form invalid UTF-8 sequences
- Reading a UTF-16 file as UTF-8 -- every other byte is `0x00`, producing garbled output

```ad-warning
title: Always Know Your File's Encoding
If you do not know the encoding of a file you are reading, you cannot reliably read it. There is no foolproof way to auto-detect encoding -- BOM detection helps but many files (especially UTF-8 files) lack a BOM. When interoperating with external systems, always agree on encoding as part of the data contract.
```

---

## Relationship to StreamReader and StreamWriter

[[StreamReader and StreamWriter]] use `Encoding` internally to convert between the byte [[Stream]] and the text your code works with.

```
Your Code ("Hello")
    ↓ writer.WriteLine("Hello")
StreamWriter
    ↓ Encoding.UTF8.GetBytes("Hello\r\n")
Stream (FileStream, MemoryStream, etc.)
    ↓ writes [72, 101, 108, 108, 111, 13, 10] to disk
```

When reading, the reverse happens:

```
Stream reads bytes [72, 101, 108, 108, 111, 13, 10]
    ↓
StreamReader
    ↓ Encoding.UTF8.GetString(bytes)
Your Code receives "Hello"
```

This is why the `Encoding` parameter in `StreamReader`/`StreamWriter` constructors matters -- it determines how the bytes-to-text conversion is performed.

---

## Getting Encodings by Name or Code Page

Beyond the static properties, you can obtain encodings by name or code page number:

```csharp
// By name
Encoding utf8     = Encoding.GetEncoding("utf-8");
Encoding shift_jis = Encoding.GetEncoding("shift_jis");   // Japanese
Encoding win1252  = Encoding.GetEncoding("windows-1252");  // Western European (Windows)

// By code page number
Encoding cp1252 = Encoding.GetEncoding(1252);
Encoding cp932  = Encoding.GetEncoding(932);  // Shift_JIS

// List all available encodings
foreach (EncodingInfo info in Encoding.GetEncodings())
{
    Console.WriteLine($"{info.CodePage}: {info.Name} -- {info.DisplayName}");
}
```

```ad-note
title: RegisterProvider for Extended Encodings (.NET Core/5+)
In .NET Core/5+, many legacy code page encodings (Shift_JIS, Windows-1252, etc.) are not available by default. You must register them first:

~~~csharp
Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
Encoding shiftJis = Encoding.GetEncoding("shift_jis"); // now works
~~~

This requires the `System.Text.Encoding.CodePages` NuGet package.
```

---

## Summary

| Concept | Detail |
|---|---|
| What Encoding does | Converts between `string`/`char` (Unicode text) and `byte[]` (raw bytes) |
| Abstract base | `System.Text.Encoding` -- access built-in encodings via static properties |
| Key methods | `GetBytes()`, `GetString()`, `GetByteCount()`, `GetCharCount()`, `GetPreamble()` |
| Default in .NET Core/5+ | UTF-8 without BOM |
| UTF-8 | Variable-width (1-4 bytes), ASCII-compatible, most common modern encoding |
| UTF-16 | .NET's internal `string` format, 2+ bytes per char, accessed via `Encoding.Unicode` |
| BOM | Byte sequence identifying encoding at file start; `GetPreamble()` returns it |
| Mojibake | Garbled text from wrong encoding -- produces garbage, not errors |
| StreamReader/StreamWriter | Use `Encoding` internally to convert between byte streams and text |
