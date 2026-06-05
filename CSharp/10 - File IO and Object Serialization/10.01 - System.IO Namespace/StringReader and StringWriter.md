---
tags:
 - csharp
 - io
 - streams
---

## What Are StringReader and StringWriter?

`StringReader` and `StringWriter` are the text equivalents of [[MemoryStream]] — they work with **in-memory strings** instead of files or network streams. They inherit from `TextReader` and `TextWriter` (the same base classes as [[StreamReader and StreamWriter|StreamReader/StreamWriter]]).

```
StreamReader / StreamWriter   → read/write text over a Stream (file, network, memory)
StringReader / StringWriter   → read/write text over a string in memory (no stream at all)
```


---

## StringReader — Read from a String

`StringReader` lets you read from a string using the same `TextReader` API (`ReadLine`, `Read`, `Peek`, `ReadToEnd`) as you would a file:

```csharp
string data = "Line 1\nLine 2\nLine 3";

using var reader = new StringReader(data);
string? line;
while ((line = reader.ReadLine()) != null)
{
    Console.WriteLine(line);
}
// Output:
// Line 1
// Line 2
// Line 3
```

### Useful Methods

| Method | Returns |
|---|---|
| `ReadLine()` | Next line (or null at end) |
| `ReadToEnd()` | Remaining text as one string |
| `Read()` | Next character as int (-1 at end) |
| `Read(char[], index, count)` | Block of characters |
| `Peek()` | Next character without consuming |


---

## StringWriter — Build a String

`StringWriter` lets you write to an internal `StringBuilder` using the `TextWriter` API (`Write`, `WriteLine`):

```csharp
using var writer = new StringWriter();
writer.WriteLine("Name: Long");
writer.WriteLine($"Date: {DateTime.Now:yyyy-MM-dd}");
writer.Write("Score: ");
writer.Write(95);

string result = writer.ToString();
Console.WriteLine(result);
// Name: Long
// Date: 2026-06-04
// Score: 95
```

### Accessing the Underlying StringBuilder

```csharp
using var writer = new StringWriter();
writer.WriteLine("Hello");

// Direct access to the StringBuilder
StringBuilder sb = writer.GetStringBuilder();
sb.Append(" — appended directly");

Console.WriteLine(writer.ToString());
// Hello
//  — appended directly
```


---

## When to Use StringReader / StringWriter

### 1. Code That Accepts TextReader / TextWriter

When a method takes a `TextReader` parameter, you can pass a `StringReader` for testing or in-memory scenarios instead of reading from a file:

```csharp
// Method that works with any TextReader
int CountLines(TextReader reader)
{
    int count = 0;
    while (reader.ReadLine() != null)
        count++;
    return count;
}

// From a file:
using var fileReader = new StreamReader("data.txt");
int fileLines = CountLines(fileReader);

// From a string (unit test, no file needed):
using var stringReader = new StringReader("a\nb\nc");
int stringLines = CountLines(stringReader);  // 3
```

### 2. Building Formatted Text

When you need to build a complex string using `Write`/`WriteLine` semantics instead of manual `StringBuilder` or string concatenation:

```csharp
string GenerateReport(List<Sale> sales)
{
    using var writer = new StringWriter();
    writer.WriteLine("Sales Report");
    writer.WriteLine("============");
    foreach (var sale in sales)
    {
        writer.WriteLine($"{sale.Date:d}  {sale.Product,-20}  {sale.Amount,10:C}");
    }
    writer.WriteLine($"Total: {sales.Sum(s => s.Amount):C}");
    return writer.ToString();
}
```

### 3. Parsing Multi-Line Strings

When you receive a multi-line string (from an API, config, etc.) and want to process it line by line:

```csharp
string csvData = httpResponse.Content;
using var reader = new StringReader(csvData);

// Skip header
reader.ReadLine();

string? line;
while ((line = reader.ReadLine()) != null)
{
    var fields = line.Split(',');
    ProcessRecord(fields);
}
```


---

## StringReader/StringWriter vs Alternatives

| Approach | Use when |
|---|---|
| `StringReader` | You need `TextReader` API on a string (line-by-line parsing, pass to methods accepting `TextReader`) |
| `string.Split('\n')` | Simple line splitting, no streaming API needed |
| `StringWriter` | You need `TextWriter` API to build a string (pass to methods accepting `TextWriter`, formatted output) |
| `StringBuilder` | Direct string building without writer semantics |
| `StreamReader` | Reading from a file or stream, not a string |

```ad-note
`StringReader`/`StringWriter` are lightweight — they're just wrappers around `string` and `StringBuilder`. No streams, no encoding, no I/O overhead. Don't hesitate to use them when the `TextReader`/`TextWriter` API is a natural fit.
```


---

## See Also

- [[StreamReader and StreamWriter]] — text I/O over streams (files, network)
- [[MemoryStream]] — the byte-level equivalent (in-memory byte array instead of string)
- [[System.IO Namespace Overview]] — full namespace taxonomy
