---
tags:
  - html
  - term
  - fundamental
---

### **🧾 SVG as XML**

SVG (**Scalable Vector Graphics**) is an **XML-based** format — that means it uses XML syntax (like HTML) to describe 2D vector graphics.

Here’s what that looks like:

---

## **🔹 Basic SVG XML Example**

```html
<?xml version="1.0" encoding="UTF-8"?>
<svg width="200" height="100" xmlns="http://www.w3.org/2000/svg">
  <rect x="10" y="10" width="180" height="80" fill="lightblue" stroke="black" stroke-width="3"/>
  <text x="100" y="60" font-size="20" text-anchor="middle" fill="black">SVG XML</text>
</svg>
```

✅ You can save this as a .svg file (e.g., example.svg) and open it in any browser.

---

## **🔹 Key XML Rules in SVG**

|**Rule**|**Example**|
|---|---|
|Must be well-formed|Every tag must be properly closed|
|Case-sensitive|\<svg> and \<SVG> are **not** the same|
|Use double quotes|Attributes like width="100"|
|Namespace required (Not sure)|Use xmlns="http://www.w3.org/2000/svg"|

---

## **🔹 Common SVG XML Elements**

|**Tag**|**Purpose**|
|---|---|
|\<svg>|Root element|
|\<rect>|Rectangle|
|\<circle>|Circle|
|\<ellipse>|Ellipse|
|\<line>|Line|
|\<path>|Complex shapes via path data|
|\<text>|Display text|
|\<g>|Group elements|

---

## **🔸 Can I Use SVG XML in HTML?**

Yes. You can **embed raw SVG XML directly in HTML** (inline), or **link to .svg files** created from XML.

```html
<img src="drawing.svg" alt="SVG Drawing" />
```

---

## **✅ Summary**

- SVG is **valid XML**, so it must follow XML syntax rules.
- You can open and edit SVG in any code editor or XML editor.
- SVG XML is widely supported and can be embedded in web pages, used as standalone graphics, or even manipulated with JavaScript.
