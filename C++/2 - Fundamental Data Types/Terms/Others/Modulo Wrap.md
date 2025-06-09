---
tags: cpp, term, fundamental
---

- Any number bigger than the largest number representable by the type simply “wraps around” (sometimes called “**modulo wrapping**”).
- For example, `255` is in range of a 1-byte integer, so `255` is fine. `256`, however, is outside the range, so it wraps around to the value `0`. `257` wraps around to the value `1`. `280` wraps around to the value `24`.