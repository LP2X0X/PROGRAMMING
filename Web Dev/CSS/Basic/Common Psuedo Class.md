1. **:hover** – Styles an element when the user hovers over it.

```css
button:hover {
  background-color: blue;
}
```

  

2. **:focus** – Styles an element when it gains focus (e.g., input fields).

```css
input:focus {
  border: 2px solid red;
}
```

  

3. **:nth-child(n)** – Targets the nth child of a parent.

```css
li:nth-child(2) {
  color: green;
}
```

  

4. **:first-child & :last-child** – Selects the first or last child of a parent.

```css
p:first-child {
  font-weight: bold;
}
```

  

5. **:not(selector)** – Excludes elements that match a selector.

```css
p:not(.special) {
  color: gray;
}
```

  

6. **:checked** – Targets checked checkboxes or radio buttons.

```css
input:checked {
  accent-color: red;
}
```

  

7. **:disabled** – Styles disabled form elements.

```css
button:disabled {
  opacity: 0.5;
}
```

  

8. **:before & :after** – Inserts content before or after an element.

```css
p::before {
  content: "★ ";
  color: gold;
}
```

