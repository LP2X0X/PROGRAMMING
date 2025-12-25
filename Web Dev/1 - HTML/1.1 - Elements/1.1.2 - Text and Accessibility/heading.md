---
tags: 
 - html
 - element
 - fundamental
---

HTML provides six heading levels:

```html
<h1>  
<h2>  
<h3>  
<h4>  
<h5>  
<h6>
```

They define the **content hierarchy**, not appearance.

---

# 🔹 **1. What Headings Actually Mean**

| Tag        | Meaning                                    |
| ---------- | ------------------------------------------ |
| **`<h1>`** | Page’s main title. Highest importance.     |
| **`<h2>`** | Major section heading.                     |
| **`<h3>`** | Subsection of `<h2>`.                      |
| **`<h4>`** | Subsection of `<h3>`.                      |
| **`<h5>`** | Usually not needed; deeper nested content. |
| **`<h6>`** | Extremely rare; almost always too deep.    |

---

# 🔹 **2. Best Practices (Important)**

## **✔ 1. Only ONE `<h1>` per page**

- Represents the main, top-level topic.
    
- In modern HTML5, multiple `<h1>`s _can_ be technically valid—but best practice is still **one per page** for clarity and SEO.
    

---

## **✔ 2. Headings should follow a logical outline**

Do this:

```
<h1>Title</h1>
  <h2>Section 1</h2>
    <h3>Subsection</h3>
  <h2>Section 2</h2>
```

❌ Don’t skip levels:

```
<h1>Title</h1>
<h3>Jumping to H3 is bad</h3>
```

---

## **✔ 3. Do NOT use headings for styling**

Bad:

```
<h3 class="big">This is actually a title</h3>
```

Use CSS for size:

```
<p class="text-4xl font-bold">Title</p>
```

Headings are for **meaning**, not **visual size**.

---

## **✔ 4. Headings improve accessibility**

Screen readers rely heavily on heading order.

- Proper structure allows users to **navigate by headings**.
    
- Wrong structure → screen reader users are lost.
    

---

## **✔ 5. Every page needs a meaningful `<h1>`**

Avoid things like:

```
<h1>Welcome!</h1>   // too generic
```

Use a descriptive topic:

```
<h1>Tailwind CSS Animation Guide</h1>
```

Better for SEO + accessibility.

---

## **✔ 6. Keep headings short, clear, and keyword-focused (SEO)**

Good:

```
<h2>How to Install Tailwind CSS</h2>
```

Bad:

```
<h2>All the things you need to know if you ever want to install Tailwind from scratch</h2>
```

---

## **✔ 7. Never wrap block elements inside headings**

❌ invalid:

```
<h2><div>Title</div></h2>
```

✔ good:

```
<h2>Title</h2>
<div>Subtitle content</div>
```

---

## **✔ 8. Don’t overuse `<h4>`, `<h5>`, `<h6>`**

- If you find yourself going deep → restructure content.
    
- HTML headings should resemble a **book outline**, not a huge nested tree.
    

---

## **✔ 9. Don’t overuse `<h4>`, `<h5>`, `<h6>`**

- Be aware that using h1-h6 tags to mark up groups of headings and their associated sub-headings/taglines is discouraged.
    
- Instead, try and reserve h1-h6 elements for when sections of content require a distinct heading. So, how should we author such content? It is recommended to group it in an [[hgroup]] element.
    

---

# 🔹 **3. Quick Cheatsheet**

**Use this mental model:**

- **H1** = page topic
    
- **H2** = main sections
    
- **H3** = subsections
    
- **Stop at H3** unless absolutely needed
    
- **No skipping levels**
    
- **No styling misuse**
    