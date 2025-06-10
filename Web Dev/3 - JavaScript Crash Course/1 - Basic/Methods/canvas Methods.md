---
tags: js, method, fundamental
---

- The **`CanvasRenderingContext2D`** object in JavaScript provides the 2D rendering context for the `<canvas>` element. This context contains a rich set of **properties** and **methods** for drawing shapes, text, images, and more.

- Here’s a detailed overview of its most important **properties and methods**:

---

## 🖌️ 1. **Shape Drawing Methods**

| Method                   | Description                          |
| ------------------------ | ------------------------------------ |
| `fillRect(x, y, w, h)`   | Draw a filled rectangle              |
| `strokeRect(x, y, w, h)` | Draw only the outline of a rectangle |
| `clearRect(x, y, w, h)`  | Clear part of the canvas             |

---

## ✏️ 2. **Path Drawing**

| Method                                                   | Description                                        |
| -------------------------------------------------------- | -------------------------------------------------- |
| `beginPath()`                                            | Start a new path                                   |
| `moveTo(x, y)`                                           | Move the pen to (x, y) without drawing             |
| `lineTo(x, y)`                                           | Draw a line to (x, y)                              |
| `arc(x, y, radius, startAngle, endAngle, anticlockwise)` | Draw a circular arc                                |
| `arcTo(x1, y1, x2, y2, radius)`                          | Draw an arc with a control point                   |
| `rect(x, y, w, h)`                                       | Add a rectangle to the path                        |
| `closePath()`                                            | Close the current path (draws back to start point) |
| `stroke()`                                               | Draw the path outline                              |
| `fill()`                                                 | Fill the interior of the path                      |

---

## 🧪 4. **Text Methods**

| Method                   | Description                                     |
| ------------------------ | ----------------------------------------------- |
| `fillText(text, x, y)`   | Draw filled text                                |
| `strokeText(text, x, y)` | Draw text outline                               |
| `measureText(text)`      | Returns a `TextMetrics` object with width, etc. |

---

## 🖼️ 5. **Image Drawing**

| Method                                             | Description                 |
| -------------------------------------------------- | --------------------------- |
| `drawImage(image, dx, dy)`                         | Draw an image at `(dx, dy)` |
| `drawImage(image, dx, dy, dw, dh)`                 | Scale image to `(dw, dh)`   |
| `drawImage(image, sx, sy, sw, sh, dx, dy, dw, dh)` | Crop and scale image        |

---

## 🔁 6. **Transformations**

| Method                        | Description                            |
| ----------------------------- | -------------------------------------- |
| `translate(x, y)`             | Move the canvas origin                 |
| `rotate(angle)`               | Rotate the canvas (in radians)         |
| `scale(x, y)`                 | Scale the canvas (zoom in/out)         |
| `transform(a, b, c, d, e, f)` | Apply a custom transformation matrix   |
| `setTransform(...)`           | Set the transformation matrix directly |
| `resetTransform()`            | Reset transformation to default        |

---

## 📋 7. **State Management**

| Method      | Description                    |
| ----------- | ------------------------------ |
| `save()`    | Save the current drawing state |
| `restore()` | Restore the last saved state   |

This lets you temporarily change styles or transformations and revert them afterward.

---

## 🧪 8. **Pixel Manipulation**

| Method                     | Description                      |
| -------------------------- | -------------------------------- |
| `getImageData(x, y, w, h)` | Get pixel data from canvas       |
| `putImageData(data, x, y)` | Put pixel data onto the canvas   |
| `createImageData(w, h)`    | Create a blank image data object |

---

## 🌀 9. **Gradients and Patterns**

| Method                                         | Description                                        |
| ---------------------------------------------- | -------------------------------------------------- |
| `createLinearGradient(x0, y0, x1, y1)`         | Create a linear gradient                           |
| `createRadialGradient(x0, y0, r0, x1, y1, r1)` | Create a radial gradient                           |
| `createPattern(image, repetition)`             | Create a pattern (e.g., `"repeat"`, `"no-repeat"`) |

---

### 📌 Example Overview

```js
const canvas = document.getElementById("myCanvas");
const ctx = canvas.getContext("2d");

ctx.fillStyle = "blue";
ctx.fillRect(10, 10, 100, 50);

ctx.font = "20px Arial";
ctx.fillText("Hello", 20, 80);
```
