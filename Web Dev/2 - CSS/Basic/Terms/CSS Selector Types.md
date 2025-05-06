- There are three primary ways you can "select" an HTML element in order to style it.
	1. By tag
	2. By class
	3. By ID


#### Example
```html
<p class="main-text" id="first-paragraph">First paragraph of document</p>
```

- Can you spot the **tag, class, and id** here? Of course! The tag is `p`, the class is `main-text`, and the ID is `first-paragraph`.

![[efdc766a3226f0ce38f86c5242294bf41acce78e-2048x1536.jpg|center]]

---

### CSS Combinators

CSS **combinators** are used to define relationships between elements and select specific elements based on their relationship in the HTML structure. There are **four main CSS combinators**:

---

### 1. **Descendant Combinator ( – Space)**

- **Selects all descendants (children, grandchildren, etc.) of a specified element.**

✅ **Example:**

```html
<div>
  <p>This is a paragraph inside a div.</p>
  <span><p>This is another paragraph.</p></span>
</div>
```

```css
div p {
  color: blue; /* Selects all <p> inside <div> */
}
```

👉 **Result:** Both `<p>` elements inside the `<div>` will be blue.

---

### 2. **Child Combinator (`>`)**

- **Selects direct children of a specified element.**

✅ **Example:**

```html
<div>
  <p>This is a direct child.</p>
  <span><p>This is NOT a direct child.</p></span>
</div>
```

```css
div > p {
  color: green; /* Selects ONLY direct child <p> */
}
```

👉 **Result:** Only the first `<p>` will be green.

---

### 3. **Adjacent Sibling Combinator (`+`)**

- **Selects the next immediate sibling (same parent, directly after).**

✅ **Example:**

```html
<h1>Heading</h1>
<p>This is the first paragraph.</p>
<p>This is the second paragraph.</p>
```

```css
h1 + p {
  color: red; /* Selects the first <p> after <h1> */
}
```

👉 **Result:** Only the **first** `<p>` after the `<h1>` will be red.

---

### 4. **General Sibling Combinator (`~`)**

- **Selects all siblings after a specified element (not just the immediate one).**

✅ **Example:**

```html
<h1>Heading</h1>
<p>Paragraph 1.</p>
<p>Paragraph 2.</p>
```

```css
h1 ~ p {
  color: purple; /* Selects ALL <p> after <h1> */
}
```

👉 **Result:** Both paragraphs after the `<h1>` will be purple.

---

### 📊 **Comparison Summary:**

|Combinator|Symbol|Selects|
|---|---|---|
|**Descendant**||All nested elements (any depth).|
|**Child**|`>`|Direct children (one level deep).|
|**Adjacent**|`+`|First immediate sibling.|
|**General**|`~`|All siblings after the element.|

---

## Selecting multiple HTML elements with CSS
- Let's say you had the following HTML.
```html
<div class="box-1">
  <p>Box 1</p>
</div>
<div class="box-2">
  <p>Box 2</p>
</div>
```
- Wouldn't it be nice if we could assign the `width` and `height` properties one time rather than writing for each element? Good news, we can! Here's how I would write this CSS.

```css
.box-1,
.box-2 {
  width: 200px;
  height: 200px;
}

.box-1 {
  border: 1px solid green;
  color: green;
}

.box-2 {
  border: 1px solid blue;
  color: blue;
}
```

- By separating each selector using a comma, we can select **multiple HTML "groups"** at the same time. In the example above, we are applying the `width` and `height` properties to _both_ selectors and then individually applying `border` and `color` (text color) styles to each.