---
tags:
  - css
  - term
  - fundamental
---
- This is a CSS Rule:
![[Pasted image 20250811103106.png|center|500]]

---

- In a CSS rule, the **last element in the selector chain** is the one that receives the styles.
	- Everything before that is just a **condition**.

- Examples:
	```css
	button:hover { background: blue; }
	```
	- Condition: button is hovered
	- Target styled: the button itself
	```css
	.card:hover .title { color: red; }
	```
	- Condition: `.card` is hovered
	- Target styled: `.title` inside `.card`
	```css
	input:checked + label { color: red; }
	```
	- Condition: input is checked
	- Target styled: the label immediately after the input