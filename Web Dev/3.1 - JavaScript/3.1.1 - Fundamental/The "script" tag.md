---
tags: js, fundamental
---

- JavaScript programs can be inserted almost anywhere into an HTML document using the `<script>` tag.
- The `<script>` tag contains JavaScript code which is automatically executed when the browser processes the tag.

---

### Externals scripts

- If we have a lot of JavaScript code, we can put it into a separate file.
- Script files are attached to HTML with the `src` attribute:

```html
<script src="/path/to/script.js"></script>
```


```ad-note
As a rule, only the simplest scripts are put into HTML. More complex ones reside in separate files.

The benefit of a separate file is that the browser will download it and store it in its [cache](https://en.wikipedia.org/wiki/Web_cache).

Other pages that reference the same script will take it from the cache instead of downloading it, so the file is actually downloaded only once.

That reduces traffic and makes pages faster.
```