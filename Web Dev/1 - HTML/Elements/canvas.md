---
tags:
  - html
  - element
  - fundamental
---

- In JavaScript, the **canvas** element is used to draw graphics on the fly — from simple shapes to interactive games, charts, or image processing. It’s part of the HTML5 spec and is controlled via JavaScript.

---

## **🧱 Basic HTML Structure**

```html
<canvas id="myCanvas" width="300" height="150"></canvas>
```

If you don’t specify a width and height, it defaults to 300×150 pixels.

---

## **🎨 Getting the Drawing Context**

You access the canvas via JavaScript and use a drawing context (usually 2D):

```js
const canvas = document.getElementById("myCanvas");
const ctx = canvas.getContext("2d"); // '2d' or 'webgl'
```

---

## **🖌️ Common Drawing Methods (2D Context)**

### **🔷 Shapes**

```js
// Rectangle
ctx.fillStyle = "blue";
ctx.fillRect(10, 10, 100, 50); // x, y, width, height

// Outline
ctx.strokeStyle = "red";
ctx.strokeRect(10, 70, 100, 50);

// Clear a section
ctx.clearRect(20, 20, 30, 30);
```

### **🧵 Paths**

```js
ctx.beginPath();
ctx.moveTo(50, 50);
ctx.lineTo(150, 50);
ctx.lineTo(100, 100);
ctx.closePath();
ctx.stroke();
ctx.fillStyle = "green";
ctx.fill();
```

### **⚪ Circles**

```js
ctx.beginPath();
ctx.arc(75, 75, 50, 0, Math.PI * 2); // x, y, radius, startAngle, endAngle
ctx.fill();
```

---

## **✍️ Text**

```js
ctx.font = "20px Arial";
ctx.fillStyle = "black";
ctx.fillText("Hello, Canvas!", 50, 50);
```

---

## **🖼️ Drawing Images**

```js
const img = new Image();
img.src = "image.png";
img.onload = function () {
  ctx.drawImage(img, 0, 0, 100, 100);
};
```

---

## **🧠 Canvas Use Cases**

- Games
- Drawing apps
- Data visualizations (e.g. with Chart.js)
- Photo/image manipulation
- Particle effects & animations
