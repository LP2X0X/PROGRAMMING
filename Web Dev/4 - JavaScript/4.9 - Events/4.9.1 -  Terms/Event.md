---
tags: 
 - js
 - term
 - event
---

- _An event_ is a signal that something has happened. All DOM nodes generate such signals (but events are not limited to DOM). Here’s a list of the most useful DOM events, just to take a look at:

- **Mouse events:**
	- `click` – when the mouse clicks on an element (touchscreen devices generate it on a tap).
	- `contextmenu` – when the mouse right-clicks on an element.
	- `mouseover` / `mouseout` – when the mouse cursor comes over / leaves an element.
	- `mousedown` / `mouseup` – when the mouse button is pressed / released over an element.
	- `mousemove` – when the mouse is moved.
- **Keyboard events:**
	- `keydown` and `keyup` – when a keyboard key is pressed and released.
- **Form element events:**
	- `submit` – when the visitor submits a `<form>`.
	- `focus` – when the visitor focuses on an element, e.g. on an `<input>`.
- **Document events:**
	- `DOMContentLoaded` – when the HTML is loaded and processed, DOM is fully built.
- **CSS events:**
	- `transitionend` – when a CSS-animation finishes.