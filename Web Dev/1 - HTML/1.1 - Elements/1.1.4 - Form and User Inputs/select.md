---
tags: 
 - html
 - element
---

## 1. Purpose of `<select>`

The `<select>` element represents a **control that allows users to choose one or more options from a predefined list**.  
It is commonly used in forms to submit structured, validated values to the server.

```html
<select name="country">
  <option value="vn">Vietnam</option>
  <option value="jp">Japan</option>
</select>
```

---

## 2. Core Structure

### Required/Typical Elements

- `<select>` – the container control
    
- `<option>` – selectable items
    
- `<optgroup>` – logical grouping of options (optional)
    

```html
<select name="fruit">
  <optgroup label="Citrus">
    <option value="orange">Orange</option>
    <option value="lemon">Lemon</option>
  </optgroup>
</select>
```

---

## 3. Important Attributes of `<select>`

### `name`

- Determines the **key used when form data is submitted**
    
- If missing, the value is **not submitted**
    

```html
<select name="size">
```

---

### `id`

- Used for label association and JS access
    

```html
<label for="size">Size</label>
<select id="size" name="size">
```

---

### `multiple`

- Allows selecting **more than one option**
    
- Submitted values are sent as multiple entries with the same name
    

```html
<select name="colors" multiple>
```

---

### `size`

- Number of visible options without scrolling
    
- If `multiple` is present, behaves like a list box
    

```html
<select name="colors" size="4">
```

---

### `disabled`

- Disables user interaction
    
- Value is **not submitted**
    

```html
<select disabled>
```

---

### `required`

- Prevents form submission if no option is selected
    

```html
<select required>
```

---

### `autofocus`

- Automatically focuses the control on page load
    

---

## 4. `<option>` Element

### Key Attributes

#### `value`

- The value submitted with the form
    
- If omitted, the option’s text is used
    

```html
<option value="small">Small</option>
```

---

#### `selected`

- Marks the default selected option
    

```html
<option selected>Default</option>
```

---

#### `disabled`

- Prevents selection
    

```html
<option disabled>Unavailable</option>
```

---

## 5. `<optgroup>`

Used to group related options for accessibility and usability.

```html
<optgroup label="Asia">
  <option>Vietnam</option>
</optgroup>
```

Rules:

- Must have a `label`
    
- Cannot be selected
    
- Improves screen-reader experience
    

---

## 6. Default Selection Behavior

- If **no option is marked `selected`**, the **first option is selected by default**
    
- In `multiple` selects, no option is selected unless specified
    

---

## 7. Form Submission Behavior

### Single select

```http
country=vn
```

### Multiple select

```http
colors=red&colors=blue
```

---

## 8. JavaScript Interaction

### Read value

```js
select.value;
```

### Read selected options

```js
select.selectedOptions;
```

### Change selection

```js
select.value = "vn";
```

---

## 9. Styling Considerations

- `<select>` styling is **browser-dependent**
    
- Some parts (dropdown arrow, menu) are not fully customizable
    
- Common approaches:
    
    - Light styling via CSS
        
    - Full replacement with custom components (e.g. React select)
        

---

## 10. Accessibility Notes

- Always associate `<label>` with `<select>`
    
- Use `<optgroup>` for large lists
    
- Avoid removing native behavior unless fully replaced accessibly
    

---

## 11. Common Pitfalls

- Missing `name` → value not submitted
    
- Expecting custom CSS behavior across browsers
    
- Forgetting `multiple` when handling arrays
    
- Using `<select>` when free text input is needed (use `<input>` instead)
    

---

## 12. Summary

- `<select>` provides controlled selection from predefined options
    
- `name` is essential for form submission
    
- `<option>` defines values; `<optgroup>` improves structure
    
- JavaScript can fully control the selection state
    
- Native `<select>` offers strong accessibility by default
    