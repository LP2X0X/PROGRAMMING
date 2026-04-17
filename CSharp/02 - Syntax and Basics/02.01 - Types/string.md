---
tags:
 - csharp
 - basics
 - types
---

## What It Is

- Alias for `System.String` (CTS type)
- A **reference type** that *behaves* like a value type
- Stored on the **heap**, but variable holds a reference on the stack
- **Immutable** — every modification creates a new string object

```csharp
string a = "hello";
string b = a;        // b points to the same object
a = a + " world";    // a now points to a NEW object; b still points to "hello"
```

---

## Immutability

This is the single most important thing to understand about strings.

```csharp
string s = "hello";
s.ToUpper();         // returns "HELLO" but s is still "hello"
s = s.ToUpper();     // now s points to the new "HELLO" object
```

**Why immutable?**
- **Thread safety** — no locks needed to share strings across threads
- **String interning** — safe to share references (see below)
- **Security** — connection strings, URLs can't be mutated after validation
- **Hashing** — hash code can be cached since content never changes

**The cost:** repeated concatenation in a loop creates many garbage objects:

```csharp
// BAD — O(n^2) allocations
string result = "";
for (int i = 0; i < 10000; i++)
    result += i.ToString();  // new string object every iteration

// GOOD — O(n)
var sb = new StringBuilder();
for (int i = 0; i < 10000; i++)
    sb.Append(i);
string result = sb.ToString();
```

---

## Memory Layout

A string object in memory:

```
[ Object Header (8 bytes) ]
[ Method Table Ptr (8 bytes) ]
[ _stringLength (4 bytes) ]     <- int, number of chars
[ _firstChar (2 bytes) ]        <- first char inline
[ ...remaining chars... ]       <- contiguous UTF-16 chars
[ null terminator (2 bytes) ]   <- for interop with native code
```

- Internally stored as **UTF-16** (`char` = 2 bytes, `System.Char`)
- `string.Length` returns the number of **`char` units**, not the number of visible characters or Unicode code points
- Supplementary Unicode characters (emoji, rare CJK) use **surrogate pairs** (2 chars):

```csharp
string emoji = "\U0001F600"; // smiley face
emoji.Length;                 // 2 (surrogate pair)
emoji.EnumerateRunes()
     .Count();               // 1 (one actual code point)
```

---

## String Interning

The CLR maintains an **intern pool** — a hash table of unique string instances.

```csharp
string a = "hello";           // goes into the intern pool at compile time
string b = "hello";           // reuses the same reference
object.ReferenceEquals(a, b); // true — same object

string c = new string("hello".ToCharArray());
object.ReferenceEquals(a, c); // false — different object, same content
a == c;                       // true — value equality

string d = string.Intern(c);
object.ReferenceEquals(a, d); // true — manually interned
```

**Rules:**
- **String literals** are automatically interned at compile time
- **Runtime-generated strings** are NOT interned (unless you call `string.Intern()`)
- Interning trades **memory** (strings never get GC'd from the pool) for **fast reference comparison**
- Use `string.IsInterned()` to check without adding to the pool

---

## String Equality — The Gotchas

```csharp
// == operator: value equality (unlike most reference types)
"hello" == "hello"    // true — operator overloaded to compare content

// Equals: same as ==
"hello".Equals("hello")    // true

// ReferenceEquals: identity comparison
object.ReferenceEquals("hello", "hello")  // true (interning)
```

**Culture-sensitive vs ordinal:**

```csharp
// DANGEROUS — culture-dependent, behavior changes by locale
"strasse".Equals("STRASSE", StringComparison.CurrentCultureIgnoreCase);
// true in German culture, potentially unexpected elsewhere

// SAFE for programmatic comparisons
"abc".Equals("ABC", StringComparison.OrdinalIgnoreCase);  // true, always

// SAFE for user-facing text
"cafe".Equals("CAFE", StringComparison.CurrentCultureIgnoreCase);
```

**The rule:** Use `Ordinal`/`OrdinalIgnoreCase` for identifiers, file paths, protocol headers. Use `CurrentCulture` for text displayed to users.

**The Turkish-I problem:**

```csharp
// In Turkish culture:
"i".ToUpper()  // returns "I" (dotted capital I), not "I"
"I".ToLower()  // returns "i" (dotless lowercase i), not "i"

// This breaks things like:
if (header.ToLower() == "content-type")  // fails in Turkish locale

// Fix:
if (header.Equals("Content-Type", StringComparison.OrdinalIgnoreCase))
```

---

## String Concatenation — What Actually Happens

```csharp
// Compiler optimizes small literal concatenation at compile time
string s = "hello" + " " + "world";  // compiled as "hello world" — single literal

// For 2-4 args, compiler uses String.Concat overloads (no array allocation)
string s = a + b;          // String.Concat(a, b)
string s = a + b + c;     // String.Concat(a, b, c)
string s = a + b + c + d; // String.Concat(a, b, c, d)

// For 5+ args, allocates a params array
string s = a + b + c + d + e;  // String.Concat(new string[] { a, b, c, d, e })
```

**Interpolation (C# 10+ with handler):**
```csharp
// No runtime parsing, no boxing, stack-allocated buffer
string s = $"Name: {name}, Age: {age}";

// Compiler generates:
var handler = new DefaultInterpolatedStringHandler(12, 2);
handler.AppendLiteral("Name: ");
handler.AppendFormatted(name);
handler.AppendLiteral(", Age: ");
handler.AppendFormatted(age);  // no boxing — generic overload
var s = handler.ToStringAndClear();
```

---

## `string.Empty` vs `""` vs `null`

```csharp
string a = "";              // interned empty string
string b = string.Empty;    // same interned instance
string c = null;            // no object at all

a.Length;    // 0
c.Length;    // NullReferenceException

string.IsNullOrEmpty(a);    // true
string.IsNullOrEmpty(c);    // true
string.IsNullOrEmpty(" ");  // false

string.IsNullOrWhiteSpace(" \t\n"); // true
```

---

## Verbatim and Raw String Literals

```csharp
// Regular — escape sequences interpreted
string path = "C:\\Users\\admin\\file.txt";

// Verbatim (@) — backslashes are literal, double-quote to escape quotes
string path = @"C:\Users\admin\file.txt";
string quote = @"He said ""hi""";

// Raw string literals (C# 11+) — no escaping needed at all
string json = """
    {
        "name": "John",
        "age": 30
    }
    """;

// Raw + interpolation
string json = $$"""
    {
        "name": "{{name}}",
        "age": {{age}}
    }
    """;
// $$ means {{ }} are the interpolation delimiters, so { } are literal
```

---

## Span and Memory-Efficient String Operations

Strings allocate heap memory. `Span<char>` and `ReadOnlySpan<char>` let you **slice without allocating**:

```csharp
string s = "hello world";

// OLD — allocates a new string
string word = s.Substring(0, 5);          // "hello" — new heap object

// NEW — zero allocation slice
ReadOnlySpan<char> word = s.AsSpan(0, 5); // view into the same memory

// Parsing without allocations
ReadOnlySpan<char> line = "123,456,789";
int first = int.Parse(line.Slice(0, 3));  // no string allocation
```

---

## StringBuilder Internals

```csharp
var sb = new StringBuilder();
```

- Maintains a **linked list of `char[]` chunks**
- Default initial capacity: 16 characters
- When a chunk fills, a new chunk is allocated (not a full copy like `List<T>`)
- `ToString()` copies all chunks into a single contiguous string

**When to use:**
- Concatenating in a **loop** (more than ~3-4 iterations)
- Building strings with **conditional** parts
- **Not** for simple 2-3 part concatenation (compiler handles that better)

---

## String and Char — Unicode Nuances

```csharp
// A char is a UTF-16 code unit, NOT a character
char c = 'A';           // U+0041, fits in one char

// Supplementary characters need two chars (surrogate pair)
string s = "\U0001D400";  // U+1D400 — Mathematical Bold Capital A
s.Length;                  // 2
s[0];                      // high surrogate
s[1];                      // low surrogate

// Rune (.NET 5+) represents a full Unicode scalar value
Rune r = new Rune(0x1D400);
r.Utf16SequenceLength;  // 2

// StringInfo for grapheme clusters (what users perceive as "one character")
string flag = "\U0001F1FA\U0001F1F8";  // US flag
flag.Length;                             // 4 (two surrogate pairs)
new StringInfo(flag).LengthInTextElements;  // 1 (one grapheme cluster)

// Combining characters
string e = "e\u0301";          // U+0065 + U+0301 (two code points)
string e2 = "\u00E9";          // U+00E9 (one code point, precomposed)
e.Length;                       // 2
e2.Length;                      // 1
e.Normalize(NormalizationForm.FormC);  // composed form — single code point
```

---

## String Comparison Performance

From fastest to slowest:

```
ReferenceEquals()          — pointer comparison
Ordinal comparison         — byte-by-byte
OrdinalIgnoreCase          — simple case mapping table
Culture-sensitive          — full Unicode collation rules
```

For dictionary keys and hash sets:

```csharp
// FAST — ordinal hash + ordinal comparison
var dict = new Dictionary<string, int>(StringComparer.Ordinal);

// SLOW — culture-aware comparison for every lookup
var dict = new Dictionary<string, int>(StringComparer.CurrentCulture);

// Case-insensitive keys (e.g., HTTP headers)
var headers = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);
```

---

## Strings Are `IEnumerable<char>`

```csharp
string s = "hello";

foreach (char c in s)
    Console.Write(c + " ");  // h e l l o

s.Where(c => char.IsUpper(c));   // LINQ works
s.Reverse();                      // IEnumerable<char>, not a string
new string(s.Reverse().ToArray()); // "olleh" — but allocates

// Careful: iterating chars breaks surrogate pairs
// Use EnumerateRunes() for Unicode-correct iteration
foreach (Rune r in s.EnumerateRunes())
    Console.Write(r);
```

---

## Pattern Matching with Strings (C# 11+)

```csharp
string input = "hello";

var result = input switch
{
    { Length: 0 } => "empty",
    { Length: > 100 } => "too long",
    ['h', ..] => "starts with h",
    [_, _, _, ..var rest] => $"rest: {new string(rest)}",
    _ => "other"
};
```

---

## SearchValues (.NET 8+)

For repeated character searches, pre-compute the lookup:

```csharp
private static readonly SearchValues<char> Vowels =
    SearchValues.Create("aeiouAEIOU");

// Uses SIMD-accelerated vectorized search
int index = "hello world".AsSpan().IndexOfAny(Vowels);
```

---

## Common Pitfalls Summary

| Pitfall | Fix |
|---|---|
| Concatenation in a loop | `StringBuilder` |
| `ToLower()` + `==` for comparison | `StringComparison.OrdinalIgnoreCase` |
| `.Length` for user-visible character count | `StringInfo.LengthInTextElements` |
| `Substring` in hot paths | `Span<char>` slicing |
| Culture-sensitive comparison for identifiers | `StringComparison.Ordinal` |
| Assuming 1 `char` = 1 character | Handle surrogate pairs / grapheme clusters |
| `string.IsNullOrEmpty` when whitespace matters | `string.IsNullOrWhiteSpace` |
| `new String(...)` expecting interning | Literals are interned; runtime strings are not |
