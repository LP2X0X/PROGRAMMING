---
tags: 
 - js
 - DOM
 - node
 - property
---

- The [innerHTML](https://w3c.github.io/DOM-Parsing/#the-innerhtml-mixin) property allows to get the HTML inside the element as a string.

- We can also modify it. So it’s one of the most powerful ways to change the page.

- If `innerHTML` inserts a `<script>` tag into the document – it becomes a part of HTML, but doesn’t execute.

```ad-note
The `innerHTML` property is only valid for element nodes.
```

---

### innerHTML +=

- We can append HTML to an element by using `elem.innerHTML+="more html"`.

It does this:

1. The old contents is removed.
2. The new `innerHTML` is written instead (a concatenation of the old and the new one).

**As the content is “zeroed-out” and rewritten from the scratch, all images and other resources will be reloaded**.

- There are other side-effects as well. For instance, if the existing text was selected with the mouse, then most browsers will remove the selection upon rewriting `innerHTML`. And if there was an `<input>` with a text entered by the visitor, then the text will be removed. And so on.