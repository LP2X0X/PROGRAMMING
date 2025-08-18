---
tags: html, element, fundamental
---

The **anchor element** in HTML is written with the `<a>` tag.  
It is used to create hyperlinks that link to another page, a section within the same page, an email, or even a file.

### Syntax

```html
<a href="URL">Link text</a>
```

### Common Attributes

- **`href`** – The destination URL (required for a working link).
    
- **`target`** – Defines where to open the link:
    
    - `_self` (default, opens in the same tab)
        
    - `_blank` (opens in a new tab/window)
        
    - `_parent`
        
    - `_top`
        
- **`title`** – Tooltip text when hovering over the link.
    
- **`download`** – Prompts to download the target file instead of navigating.
    
- **`rel`** – Specifies the relationship between the current document and the linked document (e.g., `nofollow`, `noopener`).
    

### Examples

1. **Basic link**
    

```html
<a href="https://example.com">Visit Example</a>
```

2. **Open in new tab**
    

```html
<a href="https://example.com" target="_blank">Open Example in new tab</a>
```

3. **Link to an email**
    

```html
<a href="mailto:someone@example.com">Send Email</a>
```

4. **Link to a section on the same page**
    

```html
<a href="#section1">Go to Section 1</a>

<h2 id="section1">Section 1</h2>
```