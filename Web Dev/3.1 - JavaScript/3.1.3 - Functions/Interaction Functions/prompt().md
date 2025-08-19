---
tags:
  - js
  - function
  - interactive
  - fundamental
---

- The function `prompt` accepts two arguments:
	- `title`: The text to show the visitor.
	- `default`: An optional second parameter, the initial value for the input field.

```js
result = prompt(title, [default]);
```

- It shows a modal window with a text message, an input field for the visitor, and the buttons OK/Cancel.
- The visitor can type something in the prompt input field and press OK. Then we get that text in the `result`. Or they can cancel the input by pressing Cancel or hitting the Esc key, then we get `null` as the `result`.

```ad-note
The square brackets around `default` in the syntax above denote that the parameter is optional, not required.
```

```javascript
let age = prompt('How old are you?', 100);

alert(`You are ${age} years old!`); // You are 100 years old!
```