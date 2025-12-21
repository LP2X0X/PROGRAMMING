---
tags: 
 - html
 - element
 - other
---

The `<textarea>` element is an HTML form control used to let users **input multi-line text** — unlike `<input type="text">`, which only supports single-line text.

It’s often used for:

- Comments or feedback forms
    
- Message boxes
    
- Code or description inputs
    

---

## 🧱 **Basic Syntax**

```html
<textarea name="message" rows="4" cols="50">
Default text inside the textarea.
</textarea>
```

### 🔍 Output:

A text box with 4 visible lines and 50 character width, showing default text.

---

## ⚙️ **Main Attributes**

|Attribute|Description|Example|
|---|---|---|
|`name`|Name of the field (used when submitting a form)|`<textarea name="comment"></textarea>`|
|`rows`|Number of visible text lines|`<textarea rows="5"></textarea>`|
|`cols`|Width of textarea in character columns|`<textarea cols="40"></textarea>`|
|`placeholder`|Hint text shown when empty|`<textarea placeholder="Type here..."></textarea>`|
|`disabled`|Makes textarea uneditable|`<textarea disabled></textarea>`|
|`readonly`|Text can be read but not edited|`<textarea readonly></textarea>`|
|`maxlength`|Limits number of characters|`<textarea maxlength="200"></textarea>`|
|`required`|Field must be filled before form submission|`<textarea required></textarea>`|
|`wrap`|Controls text wrapping (`soft`, `hard`, `off`)|`<textarea wrap="hard"></textarea>`|
|`autocomplete`|Enables/disables browser auto-fill|`<textarea autocomplete="on"></textarea>`|

---

## 🪶 **Text Wrapping Behavior**

|Value|Behavior|
|---|---|
|`soft` (default)|Lines wrap visually but not saved with newlines in form submission|
|`hard`|Line breaks are inserted and sent with form data|
|`off`|No wrapping; horizontal scroll appears|

---

## 🎨 **Styling with CSS**

```css
textarea {
  width: 100%;
  height: 150px;
  padding: 10px;
  resize: vertical; /* allow vertical resize only */
  font-size: 1rem;
  border-radius: 8px;
}
```

### 💡 Tip:

You can control user resizing with the `resize` property:

```css
resize: none;      /* disable resize */
resize: both;      /* allow both directions */
resize: vertical;  /* allow vertical only */
```

---

## 🧠 **Events & JavaScript Access**

|Event|When it triggers|Example|
|---|---|---|
|`oninput`|User types text|`<textarea oninput="console.log(this.value)"></textarea>`|
|`onchange`|Text changes and loses focus|`<textarea onchange="save(this.value)"></textarea>`|
|`onfocus` / `onblur`|Gain/lose focus|—|

Access the value via JavaScript:

```js
const msg = document.querySelector('#msg').value;
```

Or dynamically set value:

```js
document.querySelector('#msg').value = "Pre-filled text";
```

---

## 🧰 **Common Use Cases**

1. **User feedback form**
    
    ```html
    <label for="feedback">Your feedback:</label>
    <textarea id="feedback" name="feedback" rows="5" required></textarea>
    ```
    
2. **Comment box**
    
    ```html
    <textarea placeholder="Write your comment..."></textarea>
    ```
    
3. **Code editor (with monospace font)**
    
    ```html
    <textarea style="font-family: monospace; height: 200px;"></textarea>
    ```
    

---

## ⚡ **Accessibility Tips**

- Always use a `<label>` associated with `for` and `id`
    
- Avoid placeholder-only labels (they disappear when typing)
    
- Set a reasonable `rows` and `cols` for visibility
    

Example:

```html
<label for="bio">Short Bio:</label>
<textarea id="bio" name="bio" rows="6"></textarea>
```

---

## ✅ **Summary**

|Feature|Description|
|---|---|
|Purpose|Multi-line text input|
|Default Behavior|Scrollable, resizable, multi-line text area|
|Common Attributes|`rows`, `cols`, `placeholder`, `maxlength`, `readonly`, `required`|
|Customization|Fully stylable via CSS|
|JS Integration|Value accessible via `.value` and events like `input` and `change`|