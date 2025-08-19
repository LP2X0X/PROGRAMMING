---
tags:
  - html
  - element
  - fundamental
---

The \<path> element in SVG is one of the **most powerful and flexible** elements — it lets you draw **complex shapes** using a set of **drawing commands** (like move, line, curve, arc).

---

## **🔹 Basic Syntax**

```html
<path d="M10 10 H 90 V 90 H 10 Z" stroke="black" fill="none" />
```

- d = the **path data** (a mini language of commands)
- M, H, V, L, C, A, Z are command letters
- stroke = border color
- fill = inside color

---

## **🔹 Common Path Commands**

|**Command**|**Meaning**|**Example**|
|---|---|---|
|M x y|Move to (x, y)|M 10 10|
|L x y|Line to (x, y)|L 100 10|
|H x|Horizontal line to x|H 150|
|V y|Vertical line to y|V 50|
|C|Cubic Bézier curve|C 20 10, 50 10, 50 50|
|S|Smooth cubic Bézier|S 80 80, 100 50|
|Q|Quadratic Bézier|Q 30 10, 50 50|
|T|Smooth quadratic Bézier|T 70 90|
|A|Arc|A rx ry xrot largeArcFlag sweepFlag x y|
|Z|Close path (back to start point)|Z|

```ad-note
 The absolute commands all use uppercase letters, and the relative ones use the same letters but lowercase.
```
---

## **🔹 Example: Triangle**

```html
<svg width="200" height="100" xmlns="http://www.w3.org/2000/svg">
  <path d="M 50 10 L 90 90 L 10 90 Z" fill="lightblue" stroke="black" />
</svg>
```

This draws a triangle:

- Move to (50, 10)
- Line to (90, 90)
- Line to (10, 90)
- Close path (Z)

---

## **🔹 Example: Heart**

```html
<svg viewBox="0 0 32 29.6" xmlns="http://www.w3.org/2000/svg">
  <path d="M23.6,0c-3.1,0-5.9,1.9-7.6,4.9C14.3,1.9,11.5,0,8.4,0C3.8,0,0,3.8,0,8.4c0,7.2,8.4,11.6,16,21.2
    c7.6-9.6,16-14,16-21.2C32,3.8,28.2,0,23.6,0z"
    fill="red"/>
</svg>
```

---

## **🔸 Tips**

- You can use tools like [**SVG Path Editor**](https://yqnn.github.io/svg-path-editor/) to draw and edit paths visually.
- Use fill="none" and stroke="color" for outlines only.
- You can animate path shapes using CSS or JS for morphing effects.
