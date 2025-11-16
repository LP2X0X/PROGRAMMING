---
tags:
  - css
  - unit
  - relative
---

## **1. `em` Unit**

**Definition:**

- `em` is a **relative unit** in CSS.
    
- For the **`font-size` property**, its value is relative to the **font-size of the parent element**.
    
- For **other properties** (e.g., `width`, `height`, `padding`, `margin`), `em` is relative to the **computed font-size of the element itself**.

### **Usage Examples:**

```css
body {
  font-size: 16px;
}

p {
  font-size: 1.5em; /* 1.5 * 16px = 24px */
}

div {
  padding: 2em; /* 2 * parent font-size */
}
```

### **Characteristics:**

1. **Relative to Parent:**
    
    - The size compounds if nested: each child `em` is based on its immediate parent’s font size.
        
2. **Use Cases:**
    
    - Font scaling within a component or section.
        
    - Padding/margin relative to text size.
        
3. **Pros:**
    
    - Flexible and allows proportional scaling.
        
4. **Cons:**
    
    - Can be confusing in deeply nested elements because the size compounds.
        

```ad-tip
Using ems can be convenient when setting properties like padding, height, width, or border-radius because these scale evenly with the element if it inherits different font sizes or if the user changes the font settings.
```

### **Ems for font size together with ems for other properties**

- What makes ems tricky is when you use them for both font size and any other properties on the same element, the browser must **calculate the font size first**, and then it **uses that value to calculate the other values**. Both properties can have the same declared value, but they’ll have different computed values.

### **The shrinking font problem**
- Ems can produce unexpected results when you use them to specify the font sizes of multiple nested elements. To know the exact value for each element, you’ll need to know its inherited font size, which, if defined on the parent element in ems, requires you to know the parent element’s inherited size and so on up the tree.
- Here's an example:
```html
<body>
	<ul>
	  <li>Top level
	    <ul>
	      <li>Second level
	        <ul>
	          <li>Third level
	            <ul>
	              <li>Fourth level
	                <ul>
	                  <li>Fifth level</li>
	                </ul>
	              </li>
	            </ul>
	          </li>
	        </ul>
	      </li>
	    </ul>
	  </li>
	</ul>
</body>
```

```css
body { font-size: 16px; }

ul { font-size: 0.8em; }
```

![[Pasted image 20251116183210.png|center|500]]
 <figcaption align="center"><strong>Figure:</strong> The selector targets every ul on the page; so when these lists inherit their font size from other lists, the ems compound.</figcaption>

---

## **2. `rem` Unit**

**Definition:**

- `rem` stands for **“root em”**.
    
- Its value is always relative to the **root element’s font-size** (`<html>`), not the parent.
    

**Usage Examples:**

```css
html {
  font-size: 16px;
}

h1 {
  font-size: 2rem; /* 2 * 16px = 32px */
}

p {
  margin-bottom: 1.5rem; /* 1.5 * 16px = 24px */
}
```

**Characteristics:**

1. **Relative to Root:**
    
    - Predictable scaling, doesn’t compound with nested elements.
        
2. **Use Cases:**
    
    - Consistent typography across the site.
        
    - Spacing (margin/padding) that should remain proportional regardless of nesting.
        
3. **Pros:**
    
    - Easier to maintain than `em`.
        
    - Works well with responsive design when combined with media queries.
        
4. **Cons:**
    
    - Less flexible if you want a component’s size to scale relative to its parent.
        

---

### **Comparison Table: `em` vs `rem`**

|Feature|`em`|`rem`|
|---|---|---|
|Relative to|Parent element’s font-size|Root (`<html>`) font-size|
|Nesting|Compounds (nested elements multiply)|Does not compound|
|Best for|Component-level scaling|Global, consistent scaling|
|Example|`font-size: 1.5em;`|`font-size: 1.5rem;`|

---

**Tip:**

- Use `rem` for **global layout and typography**, and `em` for **local/component-based adjustments**.
    
- Use rems for font sizes, pixels for borders, and ems or rems for most other measures, especially paddings, margins, and border radius.
	- This way, font sizes are predictable, but you’ll still get the power of ems scaling your padding and margins should other factors alter the font size of an element.
	- Pixels make sense for borders, particularly when you want a nice fine line.