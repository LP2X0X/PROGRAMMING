- In HTML, you can apply multiple CSS classes to a single HTML element by listing them within the class attribute, separated by spaces. This allows you to combine styles from different classes.

#### Example

```html
<button class="btn primary large">Click Me</button>
```

- In the CSS:

```css
.btn {
  padding: 10px 20px;
  border-radius: 5px;
}

.primary {
  background-color: blue;
  color: white;
}

.large {
  font-size: 20px;
}
```


### How it Works:
• **Order Matters**: If multiple classes define the same property, the last one in the CSS will take precedence due to the **CSS cascade**.

• **Combine Styles**: Each class applies its styles independently, allowing you to create reusable and modular styling.

  

### Dynamic Classes in JavaScript:

- You can also dynamically add or remove classes using JavaScript.

#### Example:

```js
const button = document.querySelector('button');
button.classList.add('active'); // Adds a new class
button.classList.remove('large'); // Removes an existing class
button.classList.toggle('hidden'); // Toggles class on/off
```
