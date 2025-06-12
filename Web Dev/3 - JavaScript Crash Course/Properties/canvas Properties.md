---
tags: js, property, fundamental
---

- Here’s a detailed overview of its most important **properties**:

---

## 🎨 1. **Styling Properties**

| Property                                       | Description                                                                  |
| ---------------------------------------------- | ---------------------------------------------------------------------------- |
| `fillStyle`                                    | Color, gradient, or pattern used to fill shapes. Default: `"black"`          |
| `strokeStyle`                                  | Color, gradient, or pattern for strokes (outlines). Default: `"black"`       |
| `lineWidth`                                    | Thickness of lines. Default: `1`                                             |
| `lineCap`                                      | Style of line ends: `"butt"`, `"round"`, or `"square"`                       |
| `lineJoin`                                     | Corner style where two lines meet: `"bevel"`, `"round"`, or `"miter"`        |
| `miterLimit`                                   | Limit at which a miter is clipped to a bevel                                 |
| `globalAlpha`                                  | Opacity (0.0 to 1.0) for all drawing operations                              |
| `globalCompositeOperation`                     | Blending mode for drawing (e.g., `"source-over"`, `"lighter"`, `"multiply"`) |
| `shadowColor`                                  | Color of shadows                                                             |
| `shadowBlur`, `shadowOffsetX`, `shadowOffsetY` | Shadow effects for drawing                                                   |

---

## ✏️2. **Text's Method Properties**

| Property       | Description                                                               |
| -------------- | ------------------------------------------------------------------------- |
| `font`         | Font style (e.g., `"20px Arial"`)                                         |
| `textAlign`    | Horizontal alignment: `"left"`, `"right"`, `"center"`, `"start"`, `"end"` |
| `textBaseline` | Vertical alignment: `"top"`, `"middle"`, `"alphabetic"`, `"bottom"`       |

```ad-note
The meaning of the x-coordinate parameter for fillText depends on the text’s alignment.
For left-aligned text, the x-coordinate specifies the left edge of the text, whereas for right-aligned text it specifies the right edge (which is where and direction the text will be genereated from).
![[Pasted image 20250603102815.png|center]]
```