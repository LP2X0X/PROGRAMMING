---
tags: 
 - css
 - term
 - property
---

- Shorthand properties are properties that let you set the values of several other properties at one time. For example, font is a shorthand property that lets you set several font properties. This declaration specifies font-style, font-weight, font-size, lineheight, and font-family:

```css
font: italic bold 18px/1.2 "Helvetica", "Arial", sans-serif;
```

- Here are some other common shorthand properties:
	- background is a shorthand for multiple background properties: backgroundcolor, background-image, background-size, background-repeat, background-position, background-origin, background-clip, and background-attachment.
	- border is a shorthand for border-width, border-style, and border-color, which are each, in turn, shorthand properties as well.
	- border-width is shorthand for the top, right, bottom, and left border widths.

```ad-note
Shorthand properties are useful for keeping your code succinct and clear, but a few quirks about them aren’t readily apparent.
```

```ad-warning
Most shorthand properties let you omit certain values and only specify the bits you’re concerned with. It’s important to know, however, that doing this still sets the omitted values; they’ll be set implicitly to their initial value. This can silently override styles you specify elsewhere.
```

---

### Top, Right, Bottom, Left
- Shorthand property order particularly trips up developers when it comes to properties like *margin* and *padding* or some of the *border* properties that specify values for each of the four sides of an element. **For these properties, the values are in clockwise order, beginning at the top.**
- Remembering this order can keep you out of trouble. In fact, the word *TRouBLe* is a mnemonic you can use to remember the order: top, right, bottom, left.
- **Shorthand side values can be truncated:**
	- **1 value:** all four sides
	    
	- **2 values:** top/bottom = first, right/left = second
	    
	- **3 values:** top = first, right/left = second, bottom = third
	    
	- **4 values:** top, right, bottom, left (in that order)

---

### Horizontal, Vertical

- **Some properties only take two values**, e.g., `background-position`, `box-shadow`, `text-shadow`.  
- Unlike four-side shorthands (like `padding`), these use **horizontal first, then vertical** — just like **x, y** on a Cartesian grid.  
- Example:
	- `padding: 1em 2em` → vertical, then horizontal
	    
	- `background-position: 25% 75%` → horizontal, then vertical