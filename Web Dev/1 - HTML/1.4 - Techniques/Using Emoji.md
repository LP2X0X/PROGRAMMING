---
tags: 
 - html
 - technique
---

There are two primary approaches that work reliably across platforms when using emojis in web applications: **Unicode characters** and **emoji fonts**.

---

## 1. Unicode Characters

Emojis are defined in the Unicode standard and can be embedded directly in HTML using their Unicode code points.

### Example

The smiley face emoji 😄 has the Unicode code point:

```
U+1F604
```

It can be written in HTML as:

```html
&#x1F604;
```

### Notes

- This approach relies on the **operating system’s emoji font** (e.g., Apple Color Emoji, Segoe UI Emoji).
    
- Works well for many common emojis.
    
- Some emojis—especially newer or composite ones—may not render correctly on all platforms.
    

---

## 2. Emoji Fonts

Dedicated emoji fonts bundle emoji glyphs directly into the font itself.

Common examples:

- **Twemoji**
    
- **EmojiOne**
    

### How It Works

1. Include the emoji font in your project (via CDN or local assets).
    
2. Apply the font using CSS.
    
3. Render emojis using their Unicode character codes.
    

```css
.emoji {
  font-family: "Twemoji", "EmojiOne", sans-serif;
}
```

```html
<span class="emoji">😄</span>
```

### Advantages

- More consistent appearance across operating systems
    
- Better coverage for complex or newer emojis
    
- Reduces dependency on OS-level Unicode support
    

---

## Practical Guidance

- Unicode characters are sufficient for **simple, decorative emojis**
    
- Emoji fonts are preferable for **consistent UI rendering**
    
- Avoid relying on complex emoji sequences unless using an emoji font
    
- Always test on both macOS and Windows
    

---

## Summary

> **Unicode emojis are easy and widely supported, while emoji fonts provide better consistency and control across platforms. Choosing between them depends on how critical visual consistency is in your application.**


---

## References
https://www.w3schools.com/charsets/ref_emoji.asp
https://www.unicode.org/help/display_problems.html
https://fontawesome.com/search?q=emoji
