---
tags: 
 - js
 - function
 - fundamental
---

DOM navigation properties are great when elements are close to each other. What if they are not? How to get an arbitrary element of the page?

There are additional searching methods for that.

---

## Searching Elements: Methods & Tools

These are the main ways to find elements in the DOM.

|Method|What it searches by / what it returns|Can be called on _any_ element or must be `document`?|Live vs Static collection / special behavior|
|---|---|---|---|
|**`document.getElementById(id)`**|Returns the single element with the given `id` (must be unique)|Only on `document`|Single element (or `null` if not found) |
|**`elem.querySelector(cssSelector)`**|First element inside `elem` that matches the CSS selector|On any element (or `document`)|Static — returns one element or `null` |
|**`elem.querySelectorAll(cssSelector)`**|All elements inside `elem` matching the selector|On any element (or `document`)|Static `NodeList` (snapshot) |
|**`elem.matches(cssSelector)`**|Tests whether `elem` itself matches the selector|On any element|Returns `true`/`false` |
|**`elem.closest(cssSelector)`**|Finds the nearest ancestor (including itself) of `elem` that matches the selector|On any element|Returns the ancestor element or `null` |
|**`elem.getElementsByTagName(tagName)`**|All elements inside `elem` with the given tag (or `*` for any tag)|On any element (or `document`)|Live HTMLCollection (updates if document changes) |
|**`elem.getElementsByClassName(className)`**|All elements inside `elem` that have the given class|On any element (or `document`)|Live HTMLCollection |
|**`document.getElementsByName(name)`**|All elements in _document_ with the given `name` attribute|Only on `document`|Live collection |

````ad-note
📌 Selecting Elements: Using `id` vs `document.getElementById()`

When an element has an `id`, the browser automatically creates a **global variable** with the same name:

```html
<div id="box"></div>

<script>
  console.log(box); // the <div id="box"> element
</script>
```

This works because the browser puts element IDs into the **global scope** (the `window` object) on some pages.

> ❌ Why you should NOT use the “ID as a variable” approach

- Not reliable — does not work in strict mode, modules, or many frameworks.
    
- Can easily **collide with real variables**.
    
- Makes code unclear and “magical”.
    
- In modern JavaScript, it is considered **bad practice**.
    
- React, Vue, and modern bundlers may completely block this behavior.
    

> ✔️ Correct way (always use this)

```js
const box = document.getElementById("box");
```

This is:

- explicit
    
- predictable
    
- works in every environment
    
- avoids name collisions
    
- easy to read
    
````

---

## Important Details & Caveats

- **Uniqueness**: `id` should be unique within the document. If there are duplicates, methods depending on `id` can return unpredictable results. 
    
- **Live vs Static**:
    
    - Methods beginning with `getElementsBy…` return _live collections_ (they auto-update when the document changes). 
        
    - `querySelectorAll` returns a _static snapshot_ — it's fixed at the time you call it. 
        
- **Where you call it** matters:
    
    - `elem.querySelectorAll(...)` searches _within_ `elem`.
        
    - `document.getElementById(...)` always searches the entire document.
        
- **Selector power**: `querySelector` / `querySelectorAll` let you use **CSS selectors** 
    
- **Can use pseudo-classes as well**: Pseudo-classes in the CSS selector like `:hover` and `:active` are also supported. For instance, `document.querySelectorAll(':hover')` will return the collection with elements that the pointer is over now (in nesting order: from the outermost `<html>` to the most nested one).

---

## Usage Examples

Here are typical use-cases for each:

```html
<div id="container">
  <form name="search">
    <input name="q" type="text">
    <input type="submit">
  </form>
</div>
```

```js
// Find the form by name
let form = document.getElementsByName("search")[0]; 
// or
form = document.querySelector('form[name="search"]');

// First input in that form
let firstInput = form.querySelector('input'); 

// All inputs in that form
let inputs = form.getElementsByTagName('input');

// Element whose id is “container”
let container = document.getElementById('container');
```

----

Here’s the **best way to categorize ALL DOM search/select methods** so you can remember them cleanly 

# ⭐ **DOM Search Methods — Full Categorization**

## **1. CSS Selector-based methods (Modern)**

These use **CSS selectors** and return **static NodeLists**.

### **Single**

```js
document.querySelector(selector)
```

→ returns **first matching element** (or null)

### **Multiple**

```js
document.querySelectorAll(selector)
```

→ returns **NodeList (static)**  
→ supports `forEach`

### When to use

- Most flexible
    
- Most readable
    
- Recommended for modern JS
    

---

## **2. Legacy DOM methods**

These are older, highly performant, and return **live collections**.

### **By ID**

```js
document.getElementById(id)
```

✔️ fastest  
✔️ returns **element** (not a collection)

---

### **By Class**

```js
document.getElementsByClassName(name)
```

→ returns **HTMLCollection (live)**

---

### **By Tag**

```js
document.getElementsByTagName(name)
```

→ returns **HTMLCollection (live)**

---

### **By Name (form inputs)**

```js
document.getElementsByName(name)
```

→ returns **NodeList (live)** ← special case

---

## **3. Element-specific collections**

These don’t search by selector — they return **predefined groups** of elements.

### Document-level

```js
document.forms
document.images
document.links
document.scripts
document.anchors
```

→ **HTMLCollection (live)**

---

### Element-level

```js
element.children       // HTMLCollection (live)
element.childNodes     // NodeList (may include text nodes)
element.firstElementChild
element.lastElementChild
```

---

## **4. Tree navigation (structure search)**

(Not “selector” search, but still gets related nodes.)

```js
element.parentNode
element.previousSibling
element.nextSibling
element.closest(selector)
```

### Special one:

```js
element.closest(selector)
```

→ searches **upwards**  
→ returns **closest ancestor matching selector**

---

## **5. Shadow DOM search**

If using Web Components:

```js
element.shadowRoot.querySelector()
element.shadowRoot.querySelectorAll()
```

---

# 📌 Summary Table (Best for Memorization)

| Method                   | Returns        | Live? | Uses CSS selector? | Notes               |
| ------------------------ | -------------- | ----- | ------------------ | ------------------- |
| `querySelector`          | Element        | ❌    | ✔️                 | modern              |
| `querySelectorAll`       | NodeList       | ❌    | ✔️                 | modern              |
| `getElementById`         | Element        | ❌    | ❌                 | fastest             |
| `getElementsByClassName` | HTMLCollection | ✔️    | ❌                 | legacy              |
| `getElementsByTagName`   | HTMLCollection | ✔️    | ❌                 | legacy              |
| `getElementsByName`      | NodeList       | ⚠️ ✔️ | ❌                 | form inputs         |
| `element.children`       | HTMLCollection | ✔️    | ❌                 | elements only       |
| `childNodes`             | NodeList       | ❌    | ❌                 | includes text nodes |
| `closest`                | Element        | ❌    | ✔️                 | searches upward     |

---

If you want, I can also turn this into a **mind-map style cheat sheet**, React-focused version, or a compact flashcard version.