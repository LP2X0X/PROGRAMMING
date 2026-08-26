---
tags:
  - bit-manipulation
  - application
  - color
  - graphics
---

## Color and Graphics

Color representation in computers is one of the most practical and widespread applications of bit manipulation. Every pixel on your screen is encoded as a set of integers, and programmers routinely use shifts, masks, and bitwise operators to pack, extract, and manipulate color channels. This note covers how RGB and ARGB colors are represented at the bit level, how to pack and unpack channels using bitwise operations, and how to perform common color transformations entirely with bit manipulation.

---

## RGB Color Representation

### 24-Bit Color (True Color)

Digital displays produce colors by mixing three primary light channels: **Red**, **Green**, and **Blue**. Each channel is stored in exactly **one byte** (8 bits), giving each channel a range of `0` to `255` (see [[Bits Bytes and Words]] for byte fundamentals).

| Channel | Bit Width | Value Range | Meaning of 0      | Meaning of 255      |
|---------|-----------|-------------|--------------------|----------------------|
| Red     | 8 bits    | 0 - 255     | No red component   | Full red intensity   |
| Green   | 8 bits    | 0 - 255     | No green component | Full green intensity |
| Blue    | 8 bits    | 0 - 255     | No blue component  | Full blue intensity  |

With 8 bits per channel and 3 channels: `8 x 3 = 24 bits`, which gives us **16,777,216** possible colors (2^24). This is commonly called **24-bit color** or **true color**.

```ad-note
title: Why 8 Bits Per Channel?
8 bits (one byte) is the natural unit of storage in most architectures. 256 levels per channel is enough to fool the human eye into seeing smooth gradients in most conditions. This is also related to the concept of a [[Bits Bytes and Words|byte]] being the smallest addressable unit of memory.
```

### 32-Bit Color (ARGB / RGBA)

When transparency is needed, a fourth channel called **Alpha** is added. Alpha controls opacity:

| Alpha Value | Meaning              |
|-------------|----------------------|
| `0`         | Fully transparent    |
| `255`       | Fully opaque         |
| `1 - 254`   | Semi-transparent     |

This gives us **32-bit color**: 4 channels x 8 bits = 32 bits, which fits perfectly into a single 32-bit integer.

There are two common byte orderings:

| Format | Byte Order (MSB to LSB)      | Used By                         |
|--------|------------------------------|---------------------------------|
| ARGB   | Alpha, Red, Green, Blue      | Windows GDI+, WPF, .NET        |
| RGBA   | Red, Green, Blue, Alpha      | OpenGL, WebGL, HTML Canvas      |
| BGRA   | Blue, Green, Red, Alpha      | DirectX, some image formats     |
| ABGR   | Alpha, Blue, Green, Red      | Some embedded systems           |

```ad-warning
title: Byte Order Matters
When working across libraries or APIs, always verify the channel order. Passing an ARGB value to a function expecting RGBA will swap your red and blue channels, producing bizarre color shifts. This is one of the most common color-related bugs.
```

---

## Packing RGB into a Single Integer

### 24-Bit Packing

Instead of storing three separate bytes, we can pack R, G, and B into a **single integer** using the [[Left Shift]] and [[OR Operator]] operators:

**Formula:** `color = (r << 16) | (g << 8) | b`

Here is what happens step by step with `R=255`, `G=136`, `B=0`:

```
Step 1: Shift Red left by 16 bits
    r = 0000 0000  1111 1111                     (255)
    r << 16 = 0000 0000  1111 1111  0000 0000  0000 0000

Step 2: Shift Green left by 8 bits
    g = 0000 0000  1000 1000                     (136)
    g << 8  = 0000 0000  0000 0000  1000 1000  0000 0000

Step 3: Blue stays in place
    b = 0000 0000  0000 0000                     (0)

Step 4: OR all three together
    0000 0000  1111 1111  0000 0000  0000 0000   (r << 16)
  | 0000 0000  0000 0000  1000 1000  0000 0000   (g << 8)
  | 0000 0000  0000 0000  0000 0000  0000 0000   (b)
  --------------------------------------------------
    0000 0000  1111 1111  1000 1000  0000 0000
    = 0x00FF8800 = 16,746,496
```

The [[OR Operator]] works here because each channel occupies its own non-overlapping byte position. No bits collide, so OR effectively "merges" them together (see [[Combining and Extracting with Masks]]).

### 32-Bit ARGB Layout

```
 Bit:  31 30 29 28 27 26 25 24  23 22 21 20 19 18 17 16  15 14 13 12 11 10  9  8   7  6  5  4  3  2  1  0
       |-------- Alpha --------|  |--------- Red --------|  |------- Green ------|  |------- Blue --------|
       A  A  A  A  A  A  A  A    R  R  R  R  R  R  R  R    G  G  G  G  G  G  G  G   B  B  B  B  B  B  B  B
```

**Formula:** `color = (a << 24) | (r << 16) | (g << 8) | b`

Example with fully opaque orange (`A=255, R=255, G=136, B=0`):

```
    a << 24 = 1111 1111  0000 0000  0000 0000  0000 0000
    r << 16 = 0000 0000  1111 1111  0000 0000  0000 0000
    g << 8  = 0000 0000  0000 0000  1000 1000  0000 0000
    b       = 0000 0000  0000 0000  0000 0000  0000 0000
              -------------------------------------------------
    OR all  = 1111 1111  1111 1111  1000 1000  0000 0000
            = 0xFFFF8800
```

```ad-tip
title: Readability with Hex
Once packed, colors are much easier to read in hexadecimal than decimal. `0xFFFF8800` instantly tells you A=FF, R=FF, G=88, B=00. See [[Hexadecimal and Octal]] for the relationship between hex digits and 4-bit nibbles.
```

---

## Extracting Channels from a Packed Color

Extraction is the reverse of packing. We use the [[Right Shift]] operator to move the desired channel down to the lowest byte, then apply a **mask** of `0xFF` with the [[AND Operator]] to isolate just those 8 bits.

### Extraction Formulas

| Channel | Formula                      | Shift | Mask   |
|---------|------------------------------|-------|--------|
| Red     | `(color >> 16) & 0xFF`       | 16    | `0xFF` |
| Green   | `(color >> 8) & 0xFF`        | 8     | `0xFF` |
| Blue    | `color & 0xFF`               | 0     | `0xFF` |
| Alpha   | `(color >> 24) & 0xFF`       | 24    | `0xFF` |

### Step-by-Step: Extracting Green from 0xFFFF8800

```
color = 1111 1111  1111 1111  1000 1000  0000 0000

Step 1: Shift right by 8
    color >> 8 = 0000 0000  1111 1111  1111 1111  1000 1000

Step 2: Mask with 0xFF (see [[Creating Bit Masks]])
    0000 0000  1111 1111  1111 1111  1000 1000
  & 0000 0000  0000 0000  0000 0000  1111 1111   (0xFF)
  --------------------------------------------------
    0000 0000  0000 0000  0000 0000  1000 1000
    = 0x88 = 136
```

The mask `0xFF` is `1111 1111` in binary -- exactly 8 bits all set to 1. ANDing with this mask zeroes out everything above the lowest byte, leaving only the channel value we want. This is the standard technique described in [[Creating Bit Masks]] and [[Combining and Extracting with Masks]].

```ad-warning
title: Sign Extension Trap
In languages with signed integers (C#, Java), right-shifting a negative number (MSB = 1) uses **arithmetic shift**, which fills with 1s from the left. When extracting Alpha from an ARGB value where A >= 128, the shift may produce unexpected sign-extended bits. The `& 0xFF` mask fixes this by clearing those extra 1s, but you should be aware of why it is essential and not just decorative.

In C++, right-shifting a signed negative value is **implementation-defined behavior**. Use `unsigned int` or `uint32_t` to guarantee logical (zero-filling) shifts.
```

```ad-note
title: Why 0xFF and Not Something Else?
The mask `0xFF` equals 255, which is `2^8 - 1`. It has exactly 8 bits set to 1. Since each color channel is 8 bits wide, this mask extracts exactly one channel. If channels were 5 bits wide (as in RGB565 for 16-bit color), you would use `0x1F` (= `2^5 - 1 = 31`). The mask always matches the bit width of the field you are extracting.
```

---

## Color Manipulation with Bitwise Ops

### Color Inversion (Negative)

To invert a color (produce its photographic negative), flip every bit in the R, G, B channels using the [[XOR Operator]] with a mask that covers only those channels:

**Formula:** `inverted = color ^ 0x00FFFFFF`

```
Original:     0000 0000  1111 1111  1000 1000  0000 0000   (0x00FF8800 = orange)
XOR mask:     0000 0000  1111 1111  1111 1111  1111 1111   (0x00FFFFFF)
              -------------------------------------------------
Result:       0000 0000  0000 0000  0111 0111  1111 1111   (0x000077FF = blue-cyan)
```

The XOR mask `0x00FFFFFF` has the alpha byte as `0x00`, so alpha is left unchanged (XOR with 0 preserves the original). Each RGB bit is flipped: `255` becomes `0`, `0` becomes `255`, and `136` becomes `119`.

```ad-tip
title: Inverting with Alpha Preserved
For ARGB colors, always use `0x00FFFFFF` as the XOR mask (not `0xFFFFFFFF`). Inverting alpha would turn opaque pixels transparent and vice versa, which is almost never what you want.
```

### Brightness Adjustment by Shifting

A quick-and-dirty way to halve brightness is to right-shift each channel by 1 (dividing by 2). This must be done per-channel to avoid bits from one channel bleeding into another:

```
Half brightness:
    r = (r >> 1)
    g = (g >> 1)
    b = (b >> 1)
    darkened = (r << 16) | (g << 8) | b
```

```ad-warning
title: Shifting Whole Colors Is Wrong
You cannot simply do `color >> 1` to halve brightness. Shifting the entire packed integer causes the LSB of the red channel to spill into the MSB of the green channel. You must extract, shift, and repack each channel independently.
```

### Grayscale Conversion

A common grayscale formula weights channels by human perception:

```
gray = (r * 77 + g * 150 + b * 29) >> 8
```

The weights (`77, 150, 29`) approximate the standard luminance coefficients (0.299, 0.587, 0.114) scaled by 256. The `>> 8` at the end divides by 256 (see [[Right Shift]] for shift-as-division), avoiding slow floating-point math.

The result is a single value `0-255`. To build a grayscale pixel:

```
grayColor = (gray << 16) | (gray << 8) | gray
```

This sets R = G = B = gray, producing a neutral shade.

### Simple Alpha Blending

To blend a foreground color over a background using the alpha channel:

```
For each channel c (R, G, B):
    blended_c = (fg_c * alpha + bg_c * (255 - alpha)) / 255
```

Using bit tricks to approximate division by 255:

```
blended_c = (fg_c * alpha + bg_c * (255 - alpha) + 128) >> 8
```

The `+ 128` adds rounding. The `>> 8` divides by 256, which is very close to dividing by 255. This is a standard approximation used in software renderers.

```ad-note
title: Premultiplied Alpha
Many graphics engines use **premultiplied alpha**, where each channel is pre-multiplied by alpha/255 before storage. This simplifies blending to `result = fg + bg * (1 - alpha)` and avoids one multiplication per channel during compositing. The tradeoff is that you lose precision in the stored channel values.
```

### Channel Swapping

To swap red and blue channels (converting between RGB and BGR byte order):

```
swapped = (color & 0xFF00FF00)          // preserve alpha and green
        | ((color & 0x00FF0000) >> 16)  // move red to blue position
        | ((color & 0x000000FF) << 16)  // move blue to red position
```

This technique uses masks to isolate individual channels and shifts to reposition them, combining [[AND Operator]], [[Left Shift]], [[Right Shift]], and [[OR Operator]] in a single expression.

---

## Hex Color Codes

### Web Colors and the # Notation

In web development, colors are written as `#RRGGBB` strings (or `#AARRGGBB` for 8-digit colors with alpha). Each pair of hex digits represents one byte (see [[Hexadecimal and Octal]]):

```
#FF8800
 || || ||
 || || ++-- Blue  = 0x00 = 0
 || ++---- Green = 0x88 = 136
 ++------ Red   = 0xFF = 255
```

Since each hex digit represents exactly 4 bits ([[Binary Number System]]), two hex digits encode one byte (8 bits), and six hex digits encode three bytes (24 bits) -- the exact size of an RGB color.

### Common Colors Reference Table

| Color Name   | Hex Code    | R   | G   | B   | Binary (RGB only)                               |
|-------------|-------------|-----|-----|-----|--------------------------------------------------|
| Black       | `#000000`   | 0   | 0   | 0   | `00000000 00000000 00000000`                     |
| White       | `#FFFFFF`   | 255 | 255 | 255 | `11111111 11111111 11111111`                     |
| Red         | `#FF0000`   | 255 | 0   | 0   | `11111111 00000000 00000000`                     |
| Green       | `#00FF00`   | 0   | 255 | 0   | `00000000 11111111 00000000`                     |
| Blue        | `#0000FF`   | 0   | 0   | 255 | `00000000 00000000 11111111`                     |
| Yellow      | `#FFFF00`   | 255 | 255 | 0   | `11111111 11111111 00000000`                     |
| Cyan        | `#00FFFF`   | 0   | 255 | 255 | `00000000 11111111 11111111`                     |
| Magenta     | `#FF00FF`   | 255 | 0   | 255 | `11111111 00000000 11111111`                     |
| Orange      | `#FF8800`   | 255 | 136 | 0   | `11111111 10001000 00000000`                     |
| Gray (50%)  | `#808080`   | 128 | 128 | 128 | `10000000 10000000 10000000`                     |

```ad-tip
title: Shorthand Hex Colors in CSS
CSS allows `#RGB` shorthand where each digit is doubled: `#F80` becomes `#FF8800`. This only works when each channel's two hex digits are identical. The shorthand `#F80` is equivalent to writing `(0xF << 4 | 0xF) << 16 | (0x8 << 4 | 0x8) << 8 | (0x0 << 4 | 0x0)`.
```

---

## Code Examples

### Packing and Unpacking

#### C++

```cpp
#include <iostream>
#include <cstdint>
#include <iomanip>

// Pack RGB channels into a 24-bit color (stored in 32-bit int)
uint32_t packRGB(uint8_t r, uint8_t g, uint8_t b) {
    return (static_cast<uint32_t>(r) << 16)
         | (static_cast<uint32_t>(g) << 8)
         | static_cast<uint32_t>(b);
}

// Pack ARGB channels into a 32-bit color
uint32_t packARGB(uint8_t a, uint8_t r, uint8_t g, uint8_t b) {
    return (static_cast<uint32_t>(a) << 24)
         | (static_cast<uint32_t>(r) << 16)
         | (static_cast<uint32_t>(g) << 8)
         | static_cast<uint32_t>(b);
}

// Extract individual channels
uint8_t getRed(uint32_t color)   { return (color >> 16) & 0xFF; }
uint8_t getGreen(uint32_t color) { return (color >> 8) & 0xFF; }
uint8_t getBlue(uint32_t color)  { return color & 0xFF; }
uint8_t getAlpha(uint32_t color) { return (color >> 24) & 0xFF; }

int main() {
    uint32_t orange = packARGB(255, 255, 136, 0);

    std::cout << "Packed: 0x" << std::hex << std::uppercase
              << std::setfill('0') << std::setw(8) << orange << "\n";
    // Output: Packed: 0xFFFF8800

    std::cout << std::dec;
    std::cout << "Alpha: " << (int)getAlpha(orange) << "\n";  // 255
    std::cout << "Red:   " << (int)getRed(orange)   << "\n";  // 255
    std::cout << "Green: " << (int)getGreen(orange) << "\n";  // 136
    std::cout << "Blue:  " << (int)getBlue(orange)  << "\n";  // 0

    return 0;
}
```

```ad-warning
title: Use static_cast in C++
When shifting `uint8_t` values, always cast to `uint32_t` first. Without the cast, the compiler promotes `uint8_t` to `int` (signed, usually 32-bit), which can cause sign-extension issues if you later assign to an unsigned type. Being explicit with `static_cast<uint32_t>` avoids subtle bugs.
```

#### C\#

```csharp
using System;

public static class ColorPacker
{
    // Pack RGB into a 24-bit color (stored in int)
    public static int PackRGB(byte r, byte g, byte b)
    {
        return (r << 16) | (g << 8) | b;
    }

    // Pack ARGB into a 32-bit color
    // Use uint to avoid sign issues with alpha >= 128
    public static uint PackARGB(byte a, byte r, byte g, byte b)
    {
        return ((uint)a << 24) | ((uint)r << 16) | ((uint)g << 8) | b;
    }

    // Extract individual channels
    public static byte GetRed(uint color)   => (byte)((color >> 16) & 0xFF);
    public static byte GetGreen(uint color) => (byte)((color >> 8) & 0xFF);
    public static byte GetBlue(uint color)  => (byte)(color & 0xFF);
    public static byte GetAlpha(uint color) => (byte)((color >> 24) & 0xFF);
}

// Usage
class Program
{
    static void Main()
    {
        uint orange = ColorPacker.PackARGB(255, 255, 136, 0);

        Console.WriteLine($"Packed: 0x{orange:X8}");
        // Output: Packed: 0xFFFF8800

        Console.WriteLine($"Alpha: {ColorPacker.GetAlpha(orange)}");  // 255
        Console.WriteLine($"Red:   {ColorPacker.GetRed(orange)}");    // 255
        Console.WriteLine($"Green: {ColorPacker.GetGreen(orange)}");  // 136
        Console.WriteLine($"Blue:  {ColorPacker.GetBlue(orange)}");   // 0
    }
}
```

```ad-note
title: int vs uint in C#
When the alpha channel is 128 or above, bit 31 of the packed ARGB integer is set, making a signed `int` negative. Use `uint` for ARGB colors to avoid sign-related confusion. If interoperating with APIs that expect `int` (like `System.Drawing.Color.ToArgb()`), cast carefully and be aware that negative values are normal for opaque colors.
```

#### JavaScript

```javascript
// Pack RGB into a 24-bit color
function packRGB(r, g, b) {
    return (r << 16) | (g << 8) | b;
}

// Pack ARGB into a 32-bit color
// Use >>> 0 to force unsigned 32-bit interpretation
function packARGB(a, r, g, b) {
    return ((a << 24) | (r << 16) | (g << 8) | b) >>> 0;
}

// Extract individual channels
function getRed(color)   { return (color >> 16) & 0xFF; }
function getGreen(color) { return (color >> 8) & 0xFF; }
function getBlue(color)  { return color & 0xFF; }
function getAlpha(color) { return (color >>> 24) & 0xFF; }

// Usage
let orange = packARGB(255, 255, 136, 0);

console.log(`Packed: 0x${orange.toString(16).toUpperCase().padStart(8, '0')}`);
// Output: Packed: 0xFFFF8800

console.log(`Alpha: ${getAlpha(orange)}`);  // 255
console.log(`Red:   ${getRed(orange)}`);    // 255
console.log(`Green: ${getGreen(orange)}`);  // 136
console.log(`Blue:  ${getBlue(orange)}`);   // 0
```

```ad-warning
title: JavaScript's Signed 32-Bit Gotcha
JavaScript bitwise operators treat numbers as **signed 32-bit integers**. When alpha >= 128, the result of `(a << 24) | ...` is negative. Use `>>> 0` (unsigned right shift by 0) to convert back to an unsigned 32-bit value. For alpha extraction, use `>>>` (unsigned shift) instead of `>>` (signed shift) to avoid sign-extension filling the top bits with 1s.
```

---

### Color Manipulation Functions

#### C++

```cpp
#include <cstdint>
#include <algorithm>

// Invert RGB channels (preserve alpha)
uint32_t invertColor(uint32_t color) {
    return (color & 0xFF000000) ^ 0x00FFFFFF
         | (color & 0xFF000000);
    // Simpler equivalent:
    // return color ^ 0x00FFFFFF;
    // Works because XOR with 0 on alpha bits preserves them
}

// Convert to grayscale (luminance-weighted)
uint32_t toGrayscale(uint32_t color) {
    uint8_t r = (color >> 16) & 0xFF;
    uint8_t g = (color >> 8) & 0xFF;
    uint8_t b = color & 0xFF;
    uint8_t a = (color >> 24) & 0xFF;

    // Approximation of 0.299R + 0.587G + 0.114B
    // using fixed-point: (77R + 150G + 29B) >> 8
    uint8_t gray = static_cast<uint8_t>(
        (r * 77 + g * 150 + b * 29) >> 8
    );

    return (static_cast<uint32_t>(a) << 24)
         | (static_cast<uint32_t>(gray) << 16)
         | (static_cast<uint32_t>(gray) << 8)
         | gray;
}

// Adjust brightness (factor: 0-255 where 128 = no change)
uint32_t adjustBrightness(uint32_t color, int amount) {
    int r = std::clamp(((color >> 16) & 0xFF) + amount, 0, 255);
    int g = std::clamp(((color >> 8) & 0xFF) + amount, 0, 255);
    int b = std::clamp((color & 0xFF) + amount, 0, 255);
    uint8_t a = (color >> 24) & 0xFF;

    return (static_cast<uint32_t>(a) << 24)
         | (static_cast<uint32_t>(r) << 16)
         | (static_cast<uint32_t>(g) << 8)
         | static_cast<uint32_t>(b);
}

// Alpha blend foreground over background
uint32_t alphaBlend(uint32_t fg, uint32_t bg) {
    uint32_t alpha = (fg >> 24) & 0xFF;
    uint32_t invAlpha = 255 - alpha;

    uint32_t rFg = (fg >> 16) & 0xFF;
    uint32_t gFg = (fg >> 8) & 0xFF;
    uint32_t bFg = fg & 0xFF;

    uint32_t rBg = (bg >> 16) & 0xFF;
    uint32_t gBg = (bg >> 8) & 0xFF;
    uint32_t bBg = bg & 0xFF;

    uint32_t r = (rFg * alpha + rBg * invAlpha + 128) >> 8;
    uint32_t g = (gFg * alpha + gBg * invAlpha + 128) >> 8;
    uint32_t b = (bFg * alpha + bBg * invAlpha + 128) >> 8;

    return (0xFF << 24) | (r << 16) | (g << 8) | b;
}

// Format as hex string
std::string toHexString(uint32_t color) {
    char buf[10];
    std::snprintf(buf, sizeof(buf), "#%06X", color & 0x00FFFFFF);
    return std::string(buf);
}
```

#### C\#

```csharp
using System;

public static class ColorManipulation
{
    /// <summary>Invert RGB channels, preserving alpha.</summary>
    public static uint InvertColor(uint color)
    {
        return color ^ 0x00FFFFFF;
    }

    /// <summary>Convert to grayscale using luminance weights.</summary>
    public static uint ToGrayscale(uint color)
    {
        byte r = (byte)((color >> 16) & 0xFF);
        byte g = (byte)((color >> 8) & 0xFF);
        byte b = (byte)(color & 0xFF);
        byte a = (byte)((color >> 24) & 0xFF);

        // (77R + 150G + 29B) >> 8 approximates 0.299R + 0.587G + 0.114B
        byte gray = (byte)((r * 77 + g * 150 + b * 29) >> 8);

        return ((uint)a << 24) | ((uint)gray << 16)
             | ((uint)gray << 8) | gray;
    }

    /// <summary>Adjust brightness by adding an offset to each channel.</summary>
    public static uint AdjustBrightness(uint color, int amount)
    {
        int r = Math.Clamp(((int)(color >> 16) & 0xFF) + amount, 0, 255);
        int g = Math.Clamp(((int)(color >> 8) & 0xFF) + amount, 0, 255);
        int b = Math.Clamp(((int)color & 0xFF) + amount, 0, 255);
        byte a = (byte)((color >> 24) & 0xFF);

        return ((uint)a << 24) | ((uint)r << 16)
             | ((uint)g << 8) | (uint)b;
    }

    /// <summary>Blend foreground over background using foreground alpha.</summary>
    public static uint AlphaBlend(uint fg, uint bg)
    {
        uint alpha = (fg >> 24) & 0xFF;
        uint invAlpha = 255 - alpha;

        uint r = (((fg >> 16) & 0xFF) * alpha
               + ((bg >> 16) & 0xFF) * invAlpha + 128) >> 8;
        uint g = (((fg >> 8) & 0xFF) * alpha
               + ((bg >> 8) & 0xFF) * invAlpha + 128) >> 8;
        uint b = ((fg & 0xFF) * alpha
               + (bg & 0xFF) * invAlpha + 128) >> 8;

        return (0xFFu << 24) | (r << 16) | (g << 8) | b;
    }

    /// <summary>Format as web hex string (#RRGGBB).</summary>
    public static string ToHexString(uint color)
    {
        return $"#{color & 0x00FFFFFF:X6}";
    }
}

// Usage
class Program
{
    static void Main()
    {
        uint orange = ColorPacker.PackARGB(255, 255, 136, 0);

        uint inverted = ColorManipulation.InvertColor(orange);
        Console.WriteLine($"Inverted:   {ColorManipulation.ToHexString(inverted)}");
        // Output: Inverted:   #0077FF

        uint gray = ColorManipulation.ToGrayscale(orange);
        Console.WriteLine($"Grayscale:  {ColorManipulation.ToHexString(gray)}");
        // Output: Grayscale:  #A0A0A0

        uint brighter = ColorManipulation.AdjustBrightness(orange, 50);
        Console.WriteLine($"Brighter:   {ColorManipulation.ToHexString(brighter)}");
        // Output: Brighter:   #FFBA32

        uint semiTransparent = ColorPacker.PackARGB(128, 255, 0, 0);
        uint white = ColorPacker.PackARGB(255, 255, 255, 255);
        uint blended = ColorManipulation.AlphaBlend(semiTransparent, white);
        Console.WriteLine($"Blended:    {ColorManipulation.ToHexString(blended)}");
        // Output: Blended:    #FF8080
    }
}
```

#### JavaScript

```javascript
// Invert RGB channels, preserving alpha
function invertColor(color) {
    return ((color ^ 0x00FFFFFF) | (color & 0xFF000000)) >>> 0;
}

// Convert to grayscale using luminance weights
function toGrayscale(color) {
    const r = (color >> 16) & 0xFF;
    const g = (color >> 8) & 0xFF;
    const b = color & 0xFF;
    const a = (color >>> 24) & 0xFF;

    const gray = (r * 77 + g * 150 + b * 29) >> 8;

    return ((a << 24) | (gray << 16) | (gray << 8) | gray) >>> 0;
}

// Adjust brightness by adding an offset to each channel
function adjustBrightness(color, amount) {
    const r = Math.max(0, Math.min(255, ((color >> 16) & 0xFF) + amount));
    const g = Math.max(0, Math.min(255, ((color >> 8) & 0xFF) + amount));
    const b = Math.max(0, Math.min(255, (color & 0xFF) + amount));
    const a = (color >>> 24) & 0xFF;

    return ((a << 24) | (r << 16) | (g << 8) | b) >>> 0;
}

// Blend foreground over background using foreground alpha
function alphaBlend(fg, bg) {
    const alpha = (fg >>> 24) & 0xFF;
    const invAlpha = 255 - alpha;

    const r = (((fg >> 16) & 0xFF) * alpha
             + ((bg >> 16) & 0xFF) * invAlpha + 128) >> 8;
    const g = (((fg >> 8) & 0xFF) * alpha
             + ((bg >> 8) & 0xFF) * invAlpha + 128) >> 8;
    const b = ((fg & 0xFF) * alpha
             + (bg & 0xFF) * invAlpha + 128) >> 8;

    return ((0xFF << 24) | (r << 16) | (g << 8) | b) >>> 0;
}

// Format as web hex string (#RRGGBB)
function toHexString(color) {
    return '#' + (color & 0x00FFFFFF).toString(16).toUpperCase().padStart(6, '0');
}

// Usage
const orange = packARGB(255, 255, 136, 0);

console.log(`Inverted:  ${toHexString(invertColor(orange))}`);
// Output: Inverted:  #0077FF

console.log(`Grayscale: ${toHexString(toGrayscale(orange))}`);
// Output: Grayscale: #A0A0A0

console.log(`Brighter:  ${toHexString(adjustBrightness(orange, 50))}`);
// Output: Brighter:  #FFBA32

const semiRed = packARGB(128, 255, 0, 0);
const white   = packARGB(255, 255, 255, 255);
console.log(`Blended:   ${toHexString(alphaBlend(semiRed, white))}`);
// Output: Blended:   #FF8080
```

---

## Bitwise Operations Quick Reference for Color

| Operation             | Formula / Expression                                             | Operators Used                                      |
|-----------------------|------------------------------------------------------------------|-----------------------------------------------------|
| Pack RGB              | `(r << 16) \| (g << 8) \| b`                                    | [[Left Shift]], [[OR Operator]]                     |
| Pack ARGB             | `(a << 24) \| (r << 16) \| (g << 8) \| b`                      | [[Left Shift]], [[OR Operator]]                     |
| Extract Red           | `(color >> 16) & 0xFF`                                           | [[Right Shift]], [[AND Operator]]                   |
| Extract Green         | `(color >> 8) & 0xFF`                                            | [[Right Shift]], [[AND Operator]]                   |
| Extract Blue          | `color & 0xFF`                                                   | [[AND Operator]]                                    |
| Extract Alpha         | `(color >> 24) & 0xFF`                                           | [[Right Shift]], [[AND Operator]]                   |
| Invert RGB            | `color ^ 0x00FFFFFF`                                             | [[XOR Operator]]                                    |
| Grayscale (approx)    | `(r*77 + g*150 + b*29) >> 8`                                    | [[Right Shift]]                                     |
| Swap R and B          | `(c & 0xFF00FF00) \| ((c >> 16) & 0xFF) \| ((c & 0xFF) << 16)` | [[AND Operator]], [[Right Shift]], [[Left Shift]]   |
| Force opaque          | `color \| 0xFF000000`                                            | [[OR Operator]]                                     |
| Strip alpha           | `color & 0x00FFFFFF`                                             | [[AND Operator]], [[Creating Bit Masks]]            |

```ad-tip
title: Performance Note
Bitwise color operations are used in performance-critical code (game engines, image processing, shaders) because they avoid floating-point arithmetic entirely. Integer shifts and masks execute in a single CPU cycle on all modern processors, making them far faster than equivalent floating-point math.
```

---

## Summary

Color representation in computers maps directly onto bit manipulation fundamentals. Each color channel occupies a fixed byte position within a larger integer, and the entire workflow of packing, extracting, and transforming colors relies on the same core operators covered throughout this vault:

- **Packing** uses [[Left Shift]] to position each channel and [[OR Operator]] to merge them into a single integer without bit collisions
- **Extracting** uses [[Right Shift]] to move the target channel to the lowest byte and [[AND Operator]] with `0xFF` to isolate it (see [[Creating Bit Masks]])
- **Inversion** uses [[XOR Operator]] with `0x00FFFFFF` to flip every RGB bit while preserving alpha
- **Grayscale and blending** combine extraction, integer arithmetic, and shift-based division
- **Hex color codes** are a direct consequence of the 4-bit-to-hex-digit relationship described in [[Hexadecimal and Octal]]

Understanding these operations at the bit level -- seeing the actual 32-bit layout and knowing exactly which bits each operator touches -- is what separates someone who copies color formulas from someone who can confidently write, debug, and optimize their own.
