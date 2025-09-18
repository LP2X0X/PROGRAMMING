---
tags: 
 - js
 - function
 - fundamental
---

Here’s a structured summary of _“Searching Elements in the DOM”_ (from the [javascript.info article](https://javascript.info/searching-elements-dom)), with key methods, what they do, and when to use them.

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