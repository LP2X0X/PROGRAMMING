---
tags: 
 - js
 - DOM
 - node
 - property
---

The `textContent` provides access to the _text_ inside the element: only text, minus all `<tags>`.

For instance:

```html
<div id="news">
  <h1>Headline!</h1>
  <p>Martians attack people!</p>
</div>

<script>
  // Headline! Martians attack people!
  alert(news.textContent);
</script>
```

As we can see, only text is returned, as if all `<tags>` were cut out, but the text in them remained.

In practice, reading such text is rarely needed.

**Writing to `textContent` is much more useful, because it allows to write text the “safe way”.**

Let’s say we have an arbitrary string, for instance entered by a user, and want to show it.

- With `innerHTML` we’ll have it inserted “as HTML”, with all HTML tags.
- With `textContent` we’ll have it inserted “as text”, all symbols are treated literally.